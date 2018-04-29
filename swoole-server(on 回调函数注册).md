swoole_server->on
注册Server的事件回调函数。
>bool swoole_server->on(string $event, mixed $callback);

看下面的代码
```
$serv = new swoole_server("127.0.0.1", 9501);
$serv->on('connect', function ($serv, $fd){
    echo "Client:Connect.\n";
});
$serv->on('receive', function ($serv, $fd, $from_id, $data) {
    $serv->send($fd, 'Swoole: '.$data);
    $serv->close($fd);
});
$serv->on('close', function ($serv, $fd) {
    echo "Client: Close.\n";
});
$serv->start();

```
源码中是怎么实现的呢？

 
```
 \swoole-src-2.1.1\swoole_server.c
 PHP_METHOD(swoole_server, on)

```

# 1、  判断回调函数是否有效
 ```
     if (zend_parse_parameters(ZEND_NUM_ARGS()TSRMLS_CC, "zz", &name, &cb) == FAILURE)
    {
        return;
    }

    char *func_name = NULL;
    zend_fcall_info_cache *func_cache = emalloc(sizeof(zend_fcall_info_cache));
    if (!sw_zend_is_callable_ex(cb, NULL, 0, &func_name, NULL, func_cache, NULL TSRMLS_CC))
    {
        swoole_php_fatal_error(E_ERROR, "function '%s' is not callable", func_name);
        efree(func_name);
        return;
    }
 ```
 # 2、 根据注册事件名称把回调函数放到结构体中
 ```
  char *callback_name[PHP_SERVER_CALLBACK_NUM] = {
        "Connect",
        "Receive",
        "Close",
        "Packet",
        "Start",
        "Shutdown",
        "WorkerStart",
        "WorkerStop",
        "Task",
        "Finish",
        "WorkerExit",
        "WorkerError",
        "ManagerStart",
        "ManagerStop",
        "PipeMessage",
        NULL,
        NULL,
        NULL,
        NULL,
        "BufferFull",
        "BufferEmpty",
    };

    int i;
    char property_name[128];
    int l_property_name = 0;
    memcpy(property_name, "on", 2);

    for (i = 0; i < PHP_SERVER_CALLBACK_NUM; i++)
    {
        if (callback_name[i] == NULL)
        {
            continue;
        }
        if (strncasecmp(callback_name[i], Z_STRVAL_P(name), Z_STRLEN_P(name)) == 0)
        {
            memcpy(property_name + 2, callback_name[i], Z_STRLEN_P(name));
            l_property_name = Z_STRLEN_P(name) + 2;
            property_name[l_property_name] = '\0'; 
            //更新 swoole_server 的on+事件名称属性的值，值就是回调函数 zval 类型
            zend_update_property(swoole_server_class_entry_ptr, getThis(), property_name, l_property_name, cb TSRMLS_CC);
            php_sw_server_callbacks[i] = sw_zend_read_property(swoole_server_class_entry_ptr, getThis(), property_name, l_property_name, 0 TSRMLS_CC);
            php_sw_server_caches[i] = func_cache;
            // 回调函数指针存储内容copy 到zval 的结构体中
            sw_copy_to_stack(php_sw_server_callbacks[i], _php_sw_server_callbacks[i]);
            break;
        }
    }
    //假如注册事件没有定义，就报错。
    if (l_property_name == 0)
    {
        swoole_php_error(E_WARNING, "unknown event types[%s]", Z_STRVAL_P(name));
        efree(func_cache);
        RETURN_FALSE;
    }
 ```
 # 3、  调用swoole_server_port 监听端口对象的方法
   当注册的事件为
    Connect，Receive，Close，Packet 时就会调用swoole_server_port对象的相应方法。
    具体为何，还没看明白。
 ```

    if (i < SW_SERVER_CB_onStart)
    {
        zval *port_object = server_port_list.zobjects[0];
        zval *retval = NULL;
        sw_zval_add_ref(&port_object);
        sw_zend_call_method_with_2_params(&port_object, swoole_server_port_class_entry_ptr, NULL, "on", &retval, name, cb);
    }
    else
    {
        RETURN_TRUE;
    }
 ```







