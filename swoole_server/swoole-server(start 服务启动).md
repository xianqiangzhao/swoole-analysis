swoole_server->start 是swoole 的比较复杂的一个函数，
功能是启动服务，监听所有TCP/UDP端口。
>启动成功后会创建worker_num+2个进程。Master进程+Manager进程+serv->worker_num个Worker进程。
>启动成功后将进入事件循环，等待客户端连接请求。start方法之后的代码不会执行
>服务器关闭后，start函数返回true，并继续向下执行

有两种运行模式，一种是SWOOLE_PROCESS多进程模式，另外一种是SWOOLE_BASE基本模式。
具体说明请参照 <https://wiki.swoole.com/wiki/page/353.html>

下面来介绍一下多进程模式的启动过程。



