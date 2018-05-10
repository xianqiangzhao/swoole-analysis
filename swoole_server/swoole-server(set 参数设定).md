swoole_server->set 参数设定
>PHP_METHOD(swoole_server, set)

根据传过来的参数进行设定
```
\swoole-src-2.1.1\swoole_server.c

zval *zobject = getThis();  //取得当前object 对象
HashTable *vht;

swServer *serv = swoole_get_object(zobject); //获取serv 结构体,在__construct 中
swoole_set_object 存储进入的。

vht = Z_ARRVAL_P(zset);  //取得传递来的参数给哈希变量

```
 //chroot设定
```
   //php_swoole_array_get_value 是 sw_zend_hash_find的封装函数，最终会调用
   zend api 的 zend_hash_find来查找。找到指定的变量后放到v 中
    if (php_swoole_array_get_value(vht, "chroot", v))
    {
        convert_to_string(v);
        if (SwooleG.chroot)
        {
            sw_free(SwooleG.chroot);
        }
        SwooleG.chroot = sw_strndup(Z_STRVAL_P(v), Z_STRLEN_P(v));
    }
```
简单的设定就不说了，
看一下 cpu_affinity_ignore 设定。
官方说明
>IO密集型程序中，所有网络中断都是用CPU0来处理，如果网络IO很重，CPU0负载过高会导致网络中断无法及时处理，那网络收发包的能力就会下降。

就是通过这个参数来设置不让那个核心来处理swoole 请求。
设定类型是数组
>array('cpu_affinity_ignore' => array(0, 1))

```
 if (ignore_num >= SW_CPU_NUM)  //忽略的核心数量大于总核心数报错  

 ```
 循环核心总数，找到可用的核心并设定。
```
 for (i = 0; i < SW_CPU_NUM; i++)
        {
            flag = 1;
            SW_HASHTABLE_FOREACH_START(Z_ARRVAL_P(v), zval_core)
                int core = (int) Z_LVAL_P(zval_core);
                if (i == core)
                {
                    flag = 0;
                    break;
                }
            SW_HASHTABLE_FOREACH_END();
            if (flag)
            {
                available_cpu[available_i] = i;
                available_i++;
            }
        }
        serv->cpu_affinity_available_num = available_num;
        serv->cpu_affinity_available = available_cpu;
  }
```

最后port object 增加引用计数，调用swoole_server_port类的set 方法，
把参数传过去，进行设定。

```
  zval *port_object = server_port_list.zobjects[0];

    //增加引用计数是为了zend 垃圾回收，函数返回时会把引数减一。
    sw_zval_add_ref(&port_object);
    sw_zval_add_ref(&zset);
    
    //swoole_server_port 是增加一个监听端口就生成一个。
    //对于 swoole_server 来说，根据__construct会做出第一个port 对象。
    // port 本身有自己的set 方法，事件回调函数等等。到server的addlisten函数是再提这个话题。
    sw_zend_call_method_with_1_params(&port_object, swoole_server_port_class_entry_ptr, NULL, "set", &retval, zset);
   // php_swoole_read_init_property 是身居读写属性，如果为NULL的话就设定。
    zval *zsetting = php_swoole_read_init_property(swoole_server_class_entry_ptr, getThis(), ZEND_STRL("setting") TSRMLS_CC);
    sw_php_array_merge(Z_ARRVAL_P(zsetting), Z_ARRVAL_P(zset));
    // 销毁 zset 
    sw_zval_ptr_dtor(&zset);

```

主要是判断传递过来的zval 数组中是否有对应的key ，有的话做类型转换，取出并
赋值给SwooleG，或serv 中。
因为是zval 类型，所以调用了zend api。

 