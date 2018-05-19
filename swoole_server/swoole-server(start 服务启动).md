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
   







