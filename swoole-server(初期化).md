# 概念
我们知道swoole_server是一个比较重要的对象，
看看官方的定义
>创建一个异步服务器程序，支持TCP、UDP、UnixSocket 3种协议，支持IPv4和IPv6，支持SSL/TLS单向双向证书的隧道加密。使用者无需关注底层实现细节，仅需要设置网络事件的回调函数即可。
另外注意
>swoole_server只能用于php-cli环境，否则会抛出致命错误

还有swoole_http_server、swoole_websocket_server是他的子类。
看下面的代码
```
$serv = new swoole_server('0.0.0.0', 9501, SWOOLE_BASE, SWOOLE_SOCK_TCP);
```
这段简单的代码执行后的结果是实例化swoole_server类。
我们看看源码里面是怎么定义的。

# swoole_server __construct实例化
```
\swoole-src-2.1.1\swoole_server.c
PHP_METHOD(swoole_server, __construct)

```
大概的执行步骤
  ## 1.1 如果不是cli 模式，报错
  ```
    if (strcasecmp("cli", sapi_module.name) != 0)
    {
        swoole_php_fatal_error(E_ERROR, "swoole_server only can be used in PHP CLI mode.");
        RETURN_FALSE;
    }
 ```
  ## 1.2 判断是否已经启动，也就是不能多次启动swoole_server
   ```
    if (SwooleG.main_reactor != NULL)
    {
        swoole_php_fatal_error(E_ERROR, "eventLoop has already been created. unable to create swoole_server.");
        RETURN_FALSE;
    }
    if (SwooleGS->start > 0)
    {
        swoole_php_fatal_error(E_WARNING, "server is running. unable to create swoole_server.");
        RETURN_FALSE;
    }
  ```
   ## 1.3  分配一个server 结构体对象并初期化    
 ```
    swServer *serv = sw_malloc(sizeof (swServer));
    swServer_init(serv);
    if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "s|lll", &serv_host, &host_len, &serv_port, &serv_mode, &sock_type) == FAILURE)
    {
        swoole_php_fatal_error(E_ERROR, "invalid swoole_server parameters.");
        return;
    }
 ```
  swServer_init 就是设置默认参数
  参数内容参见 <https://wiki.swoole.com/wiki/page/13.html>
   __construct可接收的参数 是
   serv_host,serv_port,serv_mode,sock_type
   参见 <https://wiki.swoole.com/wiki/page/14.html>
 ## 1.4 根据参数建立socket  并bind
 ```
   // serv_host =SYSTEMD 的情况官方文档没有找到
  if (serv_port == 0 && strcasecmp(serv_host, "SYSTEMD") == 0)
    {
        if (swserver_add_systemd_socket(serv) <= 0)
        {
            swoole_php_fatal_error(E_ERROR, "failed to add systemd socket.");
            return;
        }
    }
    else
    {   //通常的场合是创建 socket 并返回 swListenPort对象。
        swListenPort *port = swServer_add_port(serv, sock_type, serv_host, serv_port);
        if (!port)
        {
            zend_throw_exception_ex(swoole_exception_class_entry_ptr, errno TSRMLS_CC, "failed to listen server port[%s:%d]. Error: %s[%d].",
                    serv_host, serv_port, strerror(errno), errno);
            return;
        }
    }

    

 ```
  swServer_add_port  函数
  + create socket 
    根据type 建立 socket 
    type  有 ipv4,ipv6 ,upd 模式
   ```
   \swoole-src-2.1.1\src\core\socket.c
    int swSocket_create(int type)
    
   ```
 + bind 
    ```
     \swoole-src-2.1.1\src\core\socket.c
      int swSocket_bind(int sock, int type, char *host, int *port)      
    ```
    如果port 是为0 的话，会自动分配一个端口。
    然后设置socket 为 非租塞 O_NONBLOCK & O_CLOEXEC
    最后serv 对象中增加swListenPort对象。














