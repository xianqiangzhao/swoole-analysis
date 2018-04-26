# swoole php.ini 参数初期化
```
\swoole-src-2.1.1\swoole.c

    ZEND_INIT_MODULE_GLOBALS(swoole, php_swoole_init_globals, NULL);

    //php_swoole_init_globals 默认设定
    static void php_swoole_init_globals(zend_swoole_globals *swoole_globals)
    {
      swoole_globals->aio_thread_num = SW_AIO_THREAD_NUM_DEFAULT;  //设置AIO异步文件读写的线程数量
      swoole_globals->socket_buffer_size = SW_SOCKET_BUFFER_SIZE;  //设置进程间通信的UnixSocket缓存区尺寸
      swoole_globals->display_errors = 1;//关闭/开启Swoole错误信息
      swoole_globals->use_namespace = 1;//使用命名空间类风格
      swoole_globals->use_shortname = 1;//短别名
      swoole_globals->fast_serialize = 0;//swoole_server中的task功能中是否使用swoole_serialize对异步任务数据序列化
    }


```

注意 use_namespace 这个参数没用了，默认支持两种风格了

当php.ini 中设置了就会使用php.ini 中的设定。
```

PHP_INI_BEGIN()
STD_PHP_INI_ENTRY("swoole.aio_thread_num", "2", PHP_INI_ALL, OnUpdateLong, aio_thread_num, zend_swoole_globals, swoole_globals)
STD_PHP_INI_ENTRY("swoole.display_errors", "On", PHP_INI_ALL, OnUpdateBool, display_errors, zend_swoole_globals, swoole_globals)
/**
 * namespace class style
 */
STD_PHP_INI_ENTRY("swoole.use_namespace", "On", PHP_INI_SYSTEM, OnUpdateBool, use_namespace, zend_swoole_globals, swoole_globals)
/**
 * use an short class name
 */
STD_PHP_INI_ENTRY("swoole.use_shortname", "On", PHP_INI_SYSTEM, OnUpdateBool, use_shortname, zend_swoole_globals, swoole_globals)
/**
 * enable swoole_serialize
 */
STD_PHP_INI_ENTRY("swoole.fast_serialize", "Off", PHP_INI_ALL, OnUpdateBool, fast_serialize, zend_swoole_globals, swoole_globals)
/**
 * Unix socket buffer size
 */
STD_PHP_INI_ENTRY("swoole.unixsock_buffer_size", "8388608", PHP_INI_ALL, OnUpdateLong, socket_buffer_size, zend_swoole_globals, swoole_globals)
PHP_INI_END()
```
php.ini 中设定的话
0 是关闭
1以上是打开


