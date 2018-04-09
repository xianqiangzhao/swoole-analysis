# swoole 启动后master 进程、Manager进程、Worker进程之间的关系不是太容易懂。
下面写一段代码进行验证
```
// swoole_server.php
<?php
$http = new swoole_http_server("0.0.0.0", 80); // 监听80端口
$http->set(array(    
    'worker_num' => 1,        // 启动一个worker 进程
));
$http->on('request', function ($request, $response) {
    $response->header("Content-Type", "text/html; charset=utf-8");
    $response->end("<h1>Hello Swoole. #".rand(1000, 9999)."</h1>");
});

$http->start();
```
启动
php   swoole_server.php
查看生成的进程
 
  ![image.png](https://upload-images.jianshu.io/upload_images/9076770-19192bf0d3d18328.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中  5721 是master 进程，5722是Manager进程，5724 是worker 进程
#启动三个shell 窗口 监控这三个进程
strace  -f  -p 5721   这个是master 进程，多线程所以用-f 参数
strace -p 5722
strace -p 5724
 
#打开firefox 浏览器进行请求来观察各个进程的变化
 浏览器访问 localhost 

##Manager 进程( 5722)
  ![image.png](https://upload-images.jianshu.io/upload_images/9076770-605bae27a3774bce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
阻塞在等待子进程退出事件。

##master 进程( 5721)
  ![image.png](https://upload-images.jianshu.io/upload_images/9076770-663505d0fbd8f8d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
从中可以看到
pid 为5723的线程
执行 recvfrom 系统调用 从9描述符读取数据，然后写到5描述符，
接着epoll_wait, 然后从5描述符读返回的数据。
最后写数据到9描述符。

## Worker进程(5724)
   ![image.png](https://upload-images.jianshu.io/upload_images/9076770-4da138e03d198e08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从描述符4中读GET 请求数据头，然后设置为非阻塞，调用sendto 发送
“Hello Swoole ” 到描述符4


# 总结
上面的测试正好是对swoole 官网的说明的验证
> Swoole的主线程在Accept新的连接后，会将这个连接分配给一个固定的Reactor线程，并由这个线程负责监听此socket。在socket可读时读取数据，并进行协议解析，将请求投递到Worker进程。在socket可写时将数据发送给TCP客户端。


#问题
浏览器请求后 master 进程中没有看到close socketfd 的系统调用，这个为什么呢？

把代码做如下改动
```
$http->set(array(    
    'worker_num' => 1,
    'heartbeat_check_interval' => 5,//检测间隔
    'heartbeat_idle_time' => 10   //连接超过这个时间就会close
));
```
然后浏览器访问 localhost 
strace  master 进程
过了10来秒就会看到，就会发生close

![image.png](https://upload-images.jianshu.io/upload_images/9076770-7279970389f46202.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)






