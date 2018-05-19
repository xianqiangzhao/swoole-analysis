swoole_server->start 是swoole 的比较复杂的一个函数，
功能是启动服务，监听所有TCP/UDP端口。

有两种运行模式，一种是SWOOLE_PROCESS多进程模式，另外一种是SWOOLE_BASE基本模式。
具体说明请参照 <https://wiki.swoole.com/wiki/page/353.html>

下面来大体介绍一下多进程模式的启动过程。
具体细节还是得读源码。
# 启动后的表示图
  ![ ](https://github.com/xianqiangzhao/swoole-analysis/blob/master/image/process.png?raw=true "Optional title")
 


