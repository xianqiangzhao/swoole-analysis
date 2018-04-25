# 模块初期化
swoole 也是PHP的一个普通模块，没有什么特别之处。
模块初期化，就是注册各种swoole 相关的类、函数、变量等等。
 初期化的代码在
\swoole-src-2.1.1\swoole.c
```
PHP_MINIT_FUNCTION(swoole)
```
##  1. 注册常量
```
  REGISTER_LONG_CONSTANT("SWOOLE_BASE", SW_MODE_SINGLE, CONST_CS | CONST_PERSISTENT);
  REGISTER_LONG_CONSTANT("SWOOLE_THREAD", SW_MODE_THREAD, CONST_CS | CONST_PERSISTENT);
  REGISTER_LONG_CONSTANT("SWOOLE_PROCESS", SW_MODE_PROCESS, CONST_CS | CONST_PERSISTENT);
```
##  2. 注册swoole_server类
```
 SWOOLE_INIT_CLASS_ENTRY(swoole_server_ce, "swoole_server", "Swoole\\Server", swoole_server_methods);//swoole_server_methods 是swoole_server类的方法定义结构体
    swoole_server_class_entry_ptr = zend_register_internal_class(&swoole_server_ce TSRMLS_CC);
    SWOOLE_CLASS_ALIAS(swoole_server, "Swoole\\Server");

    if (!SWOOLE_G(use_shortname))
    {
        sw_zend_hash_del(CG(function_table), ZEND_STRS("go"));
    }
    else
    {
        sw_zend_register_class_alias("Co\\Server", swoole_server_class_entry_ptr);
    }
//swoole_server_methods   具体方法介绍请参见 <https://wiki.swoole.com/wiki/page/787.html>
static zend_function_entry swoole_server_methods[] = {
    PHP_ME(swoole_server, __construct, arginfo_swoole_server__construct, ZEND_ACC_PUBLIC | ZEND_ACC_CTOR)
    PHP_ME(swoole_server, __destruct, arginfo_swoole_void, ZEND_ACC_PUBLIC | ZEND_ACC_DTOR)
    PHP_ME(swoole_server, listen, arginfo_swoole_server_listen, ZEND_ACC_PUBLIC)
    PHP_MALIAS(swoole_server, addlistener, listen, arginfo_swoole_server_listen, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, on, arginfo_swoole_server_on, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, set, arginfo_swoole_server_set_oo, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, start, arginfo_swoole_void, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, send, arginfo_swoole_server_send_oo, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, sendto, arginfo_swoole_server_sendto, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, sendwait, arginfo_swoole_server_sendwait, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, exist, arginfo_swoole_server_exist, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, protect, arginfo_swoole_server_protect, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, sendfile, arginfo_swoole_server_sendfile, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, close, arginfo_swoole_server_close, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, confirm, arginfo_swoole_server_confirm, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, pause, arginfo_swoole_server_pause, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, resume, arginfo_swoole_server_resume, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, task, arginfo_swoole_server_task, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, taskwait, arginfo_swoole_server_taskwait, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, taskWaitMulti, arginfo_swoole_server_taskWaitMulti_oo, ZEND_ACC_PUBLIC)
#ifdef SW_COROUTINE
    PHP_ME(swoole_server, taskCo, arginfo_swoole_server_taskCo, ZEND_ACC_PUBLIC)
#endif
    PHP_ME(swoole_server, finish, arginfo_swoole_server_finish_oo, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, reload, arginfo_swoole_server_reload_oo, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, shutdown, arginfo_swoole_void, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, stop, arginfo_swoole_server_stop, ZEND_ACC_PUBLIC)
    PHP_FALIAS(getLastError, swoole_last_error, arginfo_swoole_void)
    PHP_ME(swoole_server, heartbeat, arginfo_swoole_server_heartbeat_oo, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, connection_info, arginfo_swoole_connection_info, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, connection_list, arginfo_swoole_connection_list, ZEND_ACC_PUBLIC)
    //psr-0 style
    PHP_MALIAS(swoole_server, getClientInfo, connection_info, arginfo_swoole_connection_info, ZEND_ACC_PUBLIC)
    PHP_MALIAS(swoole_server, getClientList, connection_list, arginfo_swoole_connection_list, ZEND_ACC_PUBLIC)
    //timer
    PHP_FALIAS(after, swoole_timer_after, arginfo_swoole_timer_after)
    PHP_FALIAS(tick, swoole_timer_tick, arginfo_swoole_timer_tick)
    PHP_FALIAS(clearTimer, swoole_timer_clear, arginfo_swoole_timer_clear)
    PHP_FALIAS(defer, swoole_event_defer, arginfo_swoole_event_defer)
    //process
    PHP_ME(swoole_server, sendMessage, arginfo_swoole_server_sendMessage, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, addProcess, arginfo_swoole_server_addProcess, ZEND_ACC_PUBLIC)
    PHP_ME(swoole_server, stats, arginfo_swoole_void, ZEND_ACC_PUBLIC)
#ifdef SWOOLE_SOCKETS_SUPPORT
    PHP_ME(swoole_server, getSocket, arginfo_swoole_server_getSocket, ZEND_ACC_PUBLIC)
#endif
    PHP_ME(swoole_server, bind, arginfo_swoole_server_bind, ZEND_ACC_PUBLIC)
    PHP_FALIAS(__sleep, swoole_unsupport_serialize, NULL)
    PHP_FALIAS(__wakeup, swoole_unsupport_serialize, NULL)
    {NULL, NULL, NULL}
};

```
###  2.1 注册 swoole_server 类的变量
```
 zend_declare_property_null(swoole_server_class_entry_ptr, ZEND_STRL("onConnect"), ZEND_ACC_PUBLIC TSRMLS_CC);
    zend_declare_property_null(swoole_server_class_entry_ptr, ZEND_STRL("onReceive"), ZEND_ACC_PUBLIC TSRMLS_CC);
    zend_declare_property_null(swoole_server_class_entry_ptr, ZEND_STRL("onClose"), ZEND_ACC_PUBLIC TSRMLS_CC);
    zend_declare_property_null(swoole_server_class_entry_ptr, ZEND_STRL("onPacket"), ZEND_ACC_PUBLIC TSRMLS_CC);
    zend_declare_property_null(swoole_server_class_entry_ptr, ZEND_STRL("onBufferFull"), ZEND_ACC_PUBLIC TSRMLS_CC);
    zend_declare_property_null(swoole_server_class_entry_ptr, ZEND_STRL("onBufferEmpty"), ZEND_ACC_PUBLIC TSRMLS_CC);
    zend_declare_property_null(swoole_server_class_entry_ptr, ZEND_STRL("onStart"), ZEND_ACC_PUBLIC TSRMLS_CC);
    zend_declare_property_null(swoole_server_class_entry_ptr, ZEND_STRL("onShutdown"), ZEND_ACC_PUBLIC TSRMLS_CC);
 ...
```

##  3. 注册swoole_timer类

```
 SWOOLE_INIT_CLASS_ENTRY(swoole_timer_ce, "swoole_timer", "Swoole\\Timer", swoole_timer_methods);
    swoole_timer_class_entry_ptr = zend_register_internal_class(&swoole_timer_ce TSRMLS_CC);
    SWOOLE_CLASS_ALIAS(swoole_timer, "Swoole\\Timer");

static const zend_function_entry swoole_timer_methods[] =
{
    ZEND_FENTRY(tick, ZEND_FN(swoole_timer_tick), arginfo_swoole_timer_after, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    ZEND_FENTRY(after, ZEND_FN(swoole_timer_after), arginfo_swoole_timer_tick, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    ZEND_FENTRY(exists, ZEND_FN(swoole_timer_exists), arginfo_swoole_timer_exists, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    ZEND_FENTRY(clear, ZEND_FN(swoole_timer_clear), arginfo_swoole_timer_clear, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    PHP_FE_END
};

```

##  4. 注册事件定时 swoole_timer  类

```
 SWOOLE_INIT_CLASS_ENTRY(swoole_timer_ce, "swoole_timer", "Swoole\\Timer", swoole_timer_methods);
    swoole_timer_class_entry_ptr = zend_register_internal_class(&swoole_timer_ce TSRMLS_CC);
    SWOOLE_CLASS_ALIAS(swoole_timer, "Swoole\\Timer");

static const zend_function_entry swoole_timer_methods[] =
{
    ZEND_FENTRY(tick, ZEND_FN(swoole_timer_tick), arginfo_swoole_timer_after, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)   //这种定义方法是 swoole_timer->tick = 调用自定义函数 swoole_timer_tick 
    ZEND_FENTRY(after, ZEND_FN(swoole_timer_after), arginfo_swoole_timer_tick, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    ZEND_FENTRY(exists, ZEND_FN(swoole_timer_exists), arginfo_swoole_timer_exists, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    ZEND_FENTRY(clear, ZEND_FN(swoole_timer_clear), arginfo_swoole_timer_clear, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    PHP_FE_END
};

```
swoole_sever 类的tick函数定义
PHP_FALIAS(tick, swoole_timer_tick, arginfo_swoole_timer_tick)
和 swoole_timer 类的tick定义
ZEND_FENTRY(tick, ZEND_FN(swoole_timer_tick), arginfo_swoole_timer_after, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
 swoole_server->tick 和swoole_time->tick  最终都是调用
函数 swoole_timer_tick。

##  5. 注册事件循环 swoole_event  类

```
  SWOOLE_INIT_CLASS_ENTRY(swoole_event_ce, "swoole_event", "Swoole\\Event", swoole_event_methods);
    swoole_event_class_entry_ptr = zend_register_internal_class(&swoole_event_ce TSRMLS_CC);
    SWOOLE_CLASS_ALIAS(swoole_event, "Swoole\\Event");

static const zend_function_entry swoole_event_methods[] =
{
    ZEND_FENTRY(add, ZEND_FN(swoole_event_add), arginfo_swoole_event_add, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    ZEND_FENTRY(del, ZEND_FN(swoole_event_del), arginfo_swoole_event_del, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    ZEND_FENTRY(set, ZEND_FN(swoole_event_set), arginfo_swoole_event_set, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    ZEND_FENTRY(exit, ZEND_FN(swoole_event_exit), arginfo_swoole_void, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    ZEND_FENTRY(write, ZEND_FN(swoole_event_write), arginfo_swoole_event_write, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    ZEND_FENTRY(wait, ZEND_FN(swoole_event_wait), arginfo_swoole_void, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    ZEND_FENTRY(defer, ZEND_FN(swoole_event_defer), arginfo_swoole_event_defer, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    ZEND_FENTRY(cycle, ZEND_FN(swoole_event_cycle), arginfo_swoole_event_cycle, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    PHP_FE_END
};
```

##  6. 注册异步IO swoole_async  类

```
    SWOOLE_INIT_CLASS_ENTRY(swoole_async_ce, "swoole_async", "Swoole\\Async", swoole_async_methods);
    swoole_async_class_entry_ptr = zend_register_internal_class(&swoole_async_ce TSRMLS_CC);
    SWOOLE_CLASS_ALIAS(swoole_async, "Swoole\\Async");

static const zend_function_entry swoole_async_methods[] =
{
    ZEND_FENTRY(read, ZEND_FN(swoole_async_read), arginfo_swoole_async_read, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    ZEND_FENTRY(write, ZEND_FN(swoole_async_write), arginfo_swoole_async_write, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    ZEND_FENTRY(readFile, ZEND_FN(swoole_async_readfile), arginfo_swoole_async_readfile, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    ZEND_FENTRY(writeFile, ZEND_FN(swoole_async_writefile), arginfo_swoole_async_writefile, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    ZEND_FENTRY(dnsLookup, ZEND_FN(swoole_async_dns_lookup), arginfo_swoole_async_dns_lookup, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
#ifdef SW_COROUTINE
    ZEND_FENTRY(dnsLookupCoro, ZEND_FN(swoole_async_dns_lookup_coro), arginfo_swoole_async_dns_lookup_coro, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
#endif
    ZEND_FENTRY(set, ZEND_FN(swoole_async_set), arginfo_swoole_async_set, ZEND_ACC_PUBLIC | ZEND_ACC_STATIC)
    PHP_ME(swoole_async, exec, arginfo_swoole_async_exec, ZEND_ACC_PUBLIC| ZEND_ACC_STATIC)
    PHP_FE_END
};

```

##  7. 注册  swoole_connection_iterator  类  (官方文档没有找到怎么用)
```
 SWOOLE_INIT_CLASS_ENTRY(swoole_connection_iterator_ce, "swoole_connection_iterator", "Swoole\\Connection\\Iterator",  swoole_connection_iterator_methods);
    swoole_connection_iterator_class_entry_ptr = zend_register_internal_class(&swoole_connection_iterator_ce TSRMLS_CC);
    SWOOLE_CLASS_ALIAS(swoole_connection_iterator, "Swoole\\Connection\\Iterator");
    zend_class_implements(swoole_connection_iterator_class_entry_ptr TSRMLS_CC, 3, spl_ce_Iterator, spl_ce_Countable, spl_ce_ArrayAccess);

```

##  8. 注册异常  swoole_exception  类  
```
 SWOOLE_INIT_CLASS_ENTRY(swoole_exception_ce, "swoole_exception", "Swoole\\Exception", NULL);
    swoole_exception_class_entry_ptr = sw_zend_register_internal_class_ex(&swoole_exception_ce, zend_exception_get_default(TSRMLS_C), NULL TSRMLS_CC);
    SWOOLE_CLASS_ALIAS(swoole_exception, "Swoole\\Exception");
```
##  9. 最后各种初期化函数的处理

  + 共享内存和全局系统锁的初期化
    swoole_init();
    共享内存初期化 参见  <https://www.jianshu.com/p/ee798bd770f1>
  + 注册swoole_server_port 没有找到官方文档
   swoole_server_port_init(module_number TSRMLS_CC);

  + 注册swoole_client   <https://wiki.swoole.com/wiki/page/p-client.html>
   swoole_client_init(module_number TSRMLS_CC);

  + 注册 协程  Client   <https://wiki.swoole.com/wiki/page/p-coroutine_client.html>
   swoole_client_coro_init(module_number TSRMLS_CC);

+ 注册 协程  Redis   <https://wiki.swoole.com/wiki/page/589.html>
   swoole_redis_coro_init(module_number TSRMLS_CC);

 + 注册 协程  mysql   <https://wiki.swoole.com/wiki/page/p-coroutine_mysql.html>
   swoole_mysql_coro_init(module_number TSRMLS_CC);

 + 注册 协程  http_client   <https://wiki.swoole.com/wiki/page/p-coroutine_http_client.html>
   swoole_http_client_coro_init(module_number TSRMLS_CC);

 + 注册 协程 server   <https://wiki.swoole.com/wiki/page/p-coroutine.html>
   swoole_coroutine_util_init(module_number TSRMLS_CC);

  + 注册  http_client   <https://wiki.swoole.com/wiki/page/p-coroutine_http_client.html>
   swoole_http_client_init(module_number TSRMLS_CC);

   + 异步IO初期化
    swoole_async_init(module_number TSRMLS_CC);

   + 注册进程管理模块 swoole_process   <https://wiki.swoole.com/wiki/page/p-process.html>
    swoole_process_init(module_number TSRMLS_CC);

   + 注册 多进程/多线程数据共享 swoole_table <https://wiki.swoole.com/wiki/page/p-table.html>
    swoole_table_init(module_number TSRMLS_CC);

   + 注册  锁，用来实现数据同步 swoole_lock <https://wiki.swoole.com/wiki/page/p-lock.html>
    swoole_lock_init(module_number TSRMLS_CC);

   + 注册 原子计数操作 swoole_atomic <https://wiki.swoole.com/wiki/page/p-lock.html>
    swoole_atomic_init(module_number TSRMLS_CC);

  + 注册  Http服务器 swoole_http_server  <https://wiki.swoole.com/wiki/page/326.html>
    swoole_http_server_init(module_number TSRMLS_CC);

   + 注册  PHP开发者可以像C一样直接读写内存 swoole_buffer  <https://wiki.swoole.com/wiki/page/246.html>
    swoole_buffer_init(module_number TSRMLS_CC);

   + 注册  websocket  服务器 swoole_websocket_server  <https://wiki.swoole.com/wiki/page/397.html>
    swoole_websocket_init(module_number TSRMLS_CC);

   + 注册  异步MySQL客户端 swoole_mysql  <https://wiki.swoole.com/wiki/page/517.html>
    swoole_mysql_init(module_number TSRMLS_CC);

   + 注册  共享内存 swoole_mmap  <https://wiki.swoole.com/wiki/page/p-mmap.html>
    swoole_mmap_init(module_number TSRMLS_CC);

   + 注册  共享内存 swoole_channel   <https://wiki.swoole.com/wiki/page/p-channel.html>
    swoole_channel_init(module_number TSRMLS_CC);

   + 注册  协程版共享内存 swoole_channel   <https://wiki.swoole.com/wiki/page/p-coroutine_channel.html>
    swoole_channel_coro_init(module_number TSRMLS_CC);

  + 注册   swoole_ringqueue    官方文档没有
    swoole_ringqueue_init(module_number TSRMLS_CC);

   + 注册  异步Http2.0客户端 swoole_http2_client   <https://wiki.swoole.com/wiki/page/708.html>
    swoole_http2_client_init(module_number TSRMLS_CC);

   + 注册  协程版异步Http2.0客户端 swoole_http2_client   <https://wiki.swoole.com/wiki/page/856.html>
    swoole_http2_client_coro_init(module_number TSRMLS_CC);

   + 注册  高性能的序列化库 swoole_serialize   <https://wiki.swoole.com/wiki/page/p-serialize.html>
    swoole_serialize_init(module_number TSRMLS_DC);

   + 注册  异步Redis客户端 swoole_redis   <https://wiki.swoole.com/wiki/page/p-serialize.html>
    swoole_redis_init(module_number TSRMLS_CC);

   + 注册  Redis服务器端 swoole_redis_server   <https://wiki.swoole.com/wiki/page/p-redis_server.html>
    swoole_redis_server_init(module_number TSRMLS_CC);
















  
