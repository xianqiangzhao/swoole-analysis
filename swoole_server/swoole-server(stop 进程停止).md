
看看官网介绍
>使当前worker进程停止运行，并立即触发onWorkerStop回调函数。
swoole_server->stop(int $worker_id = -1, bool $waitEvent = false);

$waitEvent = true 的话可以异步退出
如果要结束其他Worker进程，需要设置$worker_id，这个worker_id 不是pid.


```
\swoole-src-2.1.1\swoole_server.c
PHP_METHOD(swoole_server, stop)
...
    zend_bool wait_reactor = 0;
    long worker_id = SwooleWG.id;  //当前worker id 

    //接受参数  workerid ,wait_reactor
    if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "|lb", &worker_id, &wait_reactor) == FAILURE)
    {
        return;
    }

     //假如当前worker_id =  全局 worker 的id 且 wait_reactor ==0 的话，设置停止变量，在worker 进程中，将会停止这个worker
    if (worker_id == SwooleWG.id && wait_reactor == 0)
    {
        SwooleG.main_reactor->running = 0;
        SwooleG.running = 0;
    }
    else
    {   //否则的话取得其它worker id， 并发送信号
        swWorker *worker = swServer_get_worker(SwooleG.serv, worker_id);
        if (worker == NULL)
        {
            RETURN_FALSE;
        }
        else if (kill(worker->pid, SIGTERM) < 0)
        {
            swoole_php_sys_error(E_WARNING, "kill(%d, SIGTERM) failed.", worker->pid);//worker 的信号处理程序swWorker_signal_handler里就会设置退出变量。
            RETURN_FALSE;
        }
    }
    RETURN_TRUE;
...

```
worker 进程是在
\swoole-src-2.1.1\src\reactor\ReactorEpoll.c 的swReactorEpoll_wait 函数里面做epoll_wait循环，
循环条件就是满足
```
 reactor->running > 0 

``` 

