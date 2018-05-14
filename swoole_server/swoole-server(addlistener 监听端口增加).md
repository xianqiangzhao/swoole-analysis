swoole_server->addlistener 监听端口增加
>PHP_METHOD(swoole_server, listen)

# addlistener 是listen的别名，代码实现还是listen
别名定义在
```
\swoole-src-2.1.1\swoole.c
PHP_MALIAS(swoole_server, addlistener, listen, arginfo_swoole_server_listen, ZEND_ACC_PUBLIC)
```

# 看一下listen的具体执行流程

```
   //接收 host , port ,type 参数
    if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "sll", &host, &host_len, &port, &sock_type) == FAILURE)
    {
        return;
    }
    //取得serv 对象
    swServer *serv = swoole_get_object(getThis());
    //增加port，并放到serv->listen_list双向链表中，到server->start时会遍历整个链表，启动listen 。

    swListenPort *ls = swServer_add_port(serv, (int) sock_type, host, (int) port);
    if (!ls)
    {
        RETURN_FALSE;
    }
    // 生成swoole_server_port 对象，并返回。就可以对这个端口设定回调函数了。
    zval *port_object = php_swoole_server_add_port(ls TSRMLS_CC);
    RETURN_ZVAL(port_object, 1, NULL);
```

具体使用例子参见官网
<https://wiki.swoole.com/wiki/page/161.html>

比如：
$p = $serv->listen("127.0.0.1", 9502, SWOOLE_SOCK_TCP);
$p->on('connect', function ($serv, $fd) {  
    echo "9502: Connect.\n";
});

当用nc localhost 9502时
 9502: Connect.就会输出到控制台。

如果不定义 $p->on('connect') 回调函数，默认使用server的回调函数。


