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
    {   //通常的场合是创建 socket 并返回 swListenPort对象。并放到serv->listen_list 链表中
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
## 1.5 更新swoole_server的host，port，mode，type等信息
  ```
    zval *server_object = getThis(); 
     ...
    zend_update_property_stringl(swoole_server_class_entry_ptr, server_object, ZEND_STRL("host"), serv_host, host_len TSRMLS_CC);
    zend_update_property_long(swoole_server_class_entry_ptr, server_object, ZEND_STRL("port"), (long) serv->listen_list->port TSRMLS_CC);
    zend_update_property_long(swoole_server_class_entry_ptr, server_object, ZEND_STRL("mode"), serv->factory_mode TSRMLS_CC);
    zend_update_property_long(swoole_server_class_entry_ptr, server_object, ZEND_STRL("type"), sock_type TSRMLS_CC);
    swoole_set_object(server_object, serv);
    ```
##  1.6 根据swoole_server中的listen_list 生成一个swoole_server_port对象
    首先swoole_server是可以监听多个端口的，也就是调用一次swoole_server->addListen（）
     就会做成一个swoole_server_port对象，这个对象存储在共享内存中，
     同时也在 server_port_list，swoole_objects结构体中保持指向。
  ```
  static zval* php_swoole_server_add_port(swListenPort *port TSRMLS_DC)
  {
       //实例化一个swoole_server_port的port_object对象。
      zval *port_object;
      SW_ALLOC_INIT_ZVAL(port_object);
      object_init_ex(port_object, swoole_server_port_class_entry_ptr);

      //server_port_list中保持对这个对象的引用
      server_port_list.zobjects[server_port_list.num++] = port_object;

      //swoole_objects中根据这个对象的hander保持着原始的swListenPort结构体 port 对象的指向。
        port 是在上一步骤中生成的socket,ip等等信息的共享内存结构体。
      swoole_server_port_property *property = emalloc(sizeof(swoole_server_port_property));
      bzero(property, sizeof(swoole_server_port_property));
      swoole_set_property(port_object, 0, property);
      swoole_set_object(port_object, port);

      //把port中的信息赋值给port_object的属性。
      zend_update_property_string(swoole_server_port_class_entry_ptr, port_object, ZEND_STRL("host"), port->host TSRMLS_CC);
      zend_update_property_long(swoole_server_port_class_entry_ptr, port_object, ZEND_STRL("port"), port->port TSRMLS_CC);
      zend_update_property_long(swoole_server_port_class_entry_ptr, port_object, ZEND_STRL("type"), port->type TSRMLS_CC);
      zend_update_property_long(swoole_server_port_class_entry_ptr, port_object, ZEND_STRL("sock"), port->sock TSRMLS_CC);

  #ifdef HAVE_PCRE
      zval *connection_iterator;
      SW_MAKE_STD_ZVAL(connection_iterator);
      object_init_ex(connection_iterator, swoole_connection_iterator_class_entry_ptr);
      zend_update_property(swoole_server_port_class_entry_ptr, port_object, ZEND_STRL("connections"), connection_iterator TSRMLS_CC);

      swConnectionIterator *i = emalloc(sizeof(swConnectionIterator));
      bzero(i, sizeof(swConnectionIterator));
      i->port = port;
      swoole_set_object(connection_iterator, i);
  #endif

      add_next_index_zval(server_port_list.zports, port_object);

      return port_object;
  }
  ```
# 最后
 swoole扩展中有各种类对象，它们之间有着千丝万缕的联系。
 还有在结构体中保持对各种信息的指向及存储。
 初期化的工作就是完成对网络的socket 打开和bind，为后续的start 做准备。







