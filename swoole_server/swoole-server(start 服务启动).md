swoole_server->start 是swoole 的比较复杂的一个函数，
功能是启动服务，监听所有TCP/UDP端口。

有两种运行模式，一种是SWOOLE_PROCESS多进程模式，另外一种是SWOOLE_BASE基本模式。
具体说明请参照 <https://wiki.swoole.com/wiki/page/353.html>

下面来大体介绍一下多进程模式的启动过程。
具体细节还是得读源码。
# 启动后的表示图
  ![ ](https://github.com/xianqiangzhao/swoole-analysis/blob/master/image/process.png?raw=true "Optional title")
 
  从中可以看到，主要有三个方面，
  ## 主进程
    主进程创建多线程来负责监听端口，接收客户端的链接。
    main 线程负责接收链接，接收到的链接分配到另外一个Reactor线程。以后就由它来负责和客户端交互了。它是负责交互数据，具体数据处理它有通过ipc 通信转发给worker 进程了。
    启动后main 线程进入soket 事件轮询。
    各个子线程在epoll 等待。最初只是加入了(worker id mod  线程数 = 当前线程id 的方式指定的worker)worker 进程通信的unix 域描述符epoll等待。
    当main 主线程有链接进来时，就会分配新的监听。这样就接管了或许的链接活动了。
  ## 管理进程
    worker进程是管理进程fork 创建的，管理进程是有master 主进程创建的。
    同样task 进程也是有管理进程创建的，管理进程负责进程管理，启动，停止等等。

  ## worker进程
   	 真正干活的苦力进程，
   	 php代码里写的回调函数onWorkerStart,onConnect,onReceive,onClose都是有这个进程负责回调的。


# 启动过程
   ## 回調函数注册
  ```
  \swoole-src-2.1.1\swoole_server.c
  php_swoole_register_callback(serv);

  ```
  ## 启动前准备
    这个过程主要是根据设定的参数，初始化线程，进程结构，链接结构等等。
    然后是Reactor创建，进程工厂模式函数注册的创建。
  ```
  \swoole-src-2.1.1\swoole_server.c
  php_swoole_server_before_start(serv, zobject TSRMLS_CC);
 
  ```
  ![ ](https://github.com/xianqiangzhao/swoole-analysis/blob/master/image/before_start.png?raw=true "Optional title")

   其中这部分非常重要，当main 主线程有链接进来是，就会执行相应的回调。
   >factory->dispatch = swFactoryProcess_dispatch;
   > factory->finish = swFactoryProcess_finish;
   > factory->start = swFactoryProcess_start;
   > factory->notify = swFactoryProcess_notify;
   > factory->shutdown = swFactoryProcess_shutdown;
   > factory->end = swFactoryProcess_end;

 ## 启动start
   ```
   \swoole-src-2.1.1\src\network\Server.c
   swServer_start(serv);

   ```
   这个函数是非常复杂的，所有的进程创建，套接字监听都在这个方法完成。
   swoole_server->start() 后就阻塞在这里了，直到服务停止。
    // start check
    检查回调函数设置是否OK，各种设置参数是否OK并进行校正。
    ```
    swServer_start_check(serv) 
   ```

    //log初期化  

    ```
    if (SwooleG.log_file)
    {
        swLog_init(SwooleG.log_file);
    }
    ```

   //如果设定启动守护模式,进入守护模式

    ```
    if (daemon(0, 1) < 0)
        {
            return SW_ERR;
        }
    ```

   // master  pid 设定

	```
    SwooleGS->master_pid = getpid();
    SwooleGS->now = SwooleStats->start_time = time(NULL);
	```

   // worker 进程共享内存结构分配   
    因为要对所有进程透明，所以要从共享内存上申请内存

    ```
       serv->workers = SwooleG.memory_pool->alloc(SwooleG.memory_pool, serv->worker_num * sizeof(swWorker));
	```

   //store to swProcessPool object
    processpoll 是用来统一管理进程的结构体

    ```
    SwooleGS->event_workers.workers = serv->workers;
    SwooleGS->event_workers.worker_num = serv->worker_num;
	for (i = 0; i < serv->worker_num; i++)
    {
        SwooleGS->event_workers.workers[i].pool = &SwooleGS->event_workers;
    }
    ```

    //factory start
    这个方法是核心的一个，manger 进程，worker 进程都阻塞在这里，不返回。

    ```
    if (factory->start(factory) < 0) // swFactoryProcess_start
    {
        return SW_ERR;
    }
    ```
    
    //主进程 信号初期化signal Init

    ```
    swServer_signal_init(serv);
	```

    // 创建线程，事件循环，不返回。

    ```
    if (serv->factory_mode == SW_MODE_SINGLE) //基本模式
    {
        ret = swReactorProcess_start(serv);
    }
    else
    {
        ret = swServer_start_proxy(serv);  //多进程模式，默认会执行这个函数
    }
   ```

   本篇就说这么多，没有办法对各个函数一一展开。
   接下来，会分别说明各个进程创建的过程。
   * 1、manager 创建
   * 2、worker 创建
   * 3、task 创建
   * 4、线程创建



