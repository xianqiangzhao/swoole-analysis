#swoole http server 简单程序生成的opcode 
##代码如下
```
<?php
$http = new swoole_http_server("0.0.0.0", 8080);
$http->set(array(    
    'worker_num' => 1
));
$http->on('request', function ($request, $response) {
    //var_dump($request->get, $request->post);
    $response->header("Connection: close","Content-Type", "text/html; charset=utf-8");
    $response->end("<h1>Hello Swoole. #".rand(1000, 9999)."</h1>");
});
$http->on('close', function ($serv, $fd) {
    echo "Client: Close{$fd}.\n";
});
$http->start();
```
## 查看opcode 
```
php -dvld.active=1 swoole_http_server.php
```
当然 dvld 扩展需要提前安装

```
Finding entry points
Branch analysis from position: 0
Jump found. (Code = 62) Position 1 = -2
filename:       /home/svn/php-traning/swoole_http_server.php
function name:  (null)
number of ops:  21
compiled vars:  !0 = $http
line     #* E I O op                           fetch          ext  return  operands
-------------------------------------------------------------------------------------
   2     0  E >   NEW                                              $1      :-5                    //new swoole_http_server
         1        SEND_VAL_EX                                              '0.0.0.0'        //参数1
         2        SEND_VAL_EX                                              8080            //参数2
         3        DO_FCALL                                      0                                 //swoole_http_server类初始化？
         4        ASSIGN                                                   !0, $1                  // $http 赋值
   3     5        INIT_METHOD_CALL                                         !0, 'set'  //调用set 函数
   4     6        SEND_VAL_EX                                              <array>
         7        DO_FCALL                                      0
   6     8        INIT_METHOD_CALL                                         !0, 'on'
         9        SEND_VAL_EX                                              'request'
        10        DECLARE_LAMBDA_FUNCTION                                  '%00%7Bclosure%7D%2Fhome%2Fsvn%2Fphp-traning%2Fswoole_http_server.php0x7f162b299167'                    //匿名函数
  10    11        SEND_VAL_EX                                              ~5
        12        DO_FCALL                                      0
  11    13        INIT_METHOD_CALL                                         !0, 'on'
        14        SEND_VAL_EX                                              'close'
        15        DECLARE_LAMBDA_FUNCTION                                  '%00%7Bclosure%7D%2Fhome%2Fsvn%2Fphp-traning%2Fswoole_http_server.php0x7f162b2991b8'
  13    16        SEND_VAL_EX                                              ~7
        17        DO_FCALL                                      0
  14    18        INIT_METHOD_CALL                                         !0, 'start'
        19        DO_FCALL                                      0
  15    20      > RETURN                                                   1
//下面是 http->on('request'  的匿名函数的opcode 
branch: #  0; line:     2-   15; sop:     0; eop:    20; out1:  -2
path #1: 0,
Function %00%7Bclosure%7D%2Fhome%2Fsvn%2Fphp-traning%2Fswoole_http_server.php0x7f162b299167:
Finding entry points
Branch analysis from position: 0
Jump found. (Code = 62) Position 1 = -2
filename:       /home/svn/php-traning/swoole_http_server.php
function name:  {closure}
number of ops:  17
compiled vars:  !0 = $request, !1 = $response
line     #* E I O op                           fetch          ext  return  operands
-------------------------------------------------------------------------------------
   6     0  E >   RECV                                             !0                          //接受参数 
         1        RECV                                             !1
   8     2        INIT_METHOD_CALL                                         !1, 'header'    
         3        SEND_VAL_EX                                              'Connection%3A+close'
         4        SEND_VAL_EX                                              'Content-Type'
         5        SEND_VAL_EX                                              'text%2Fhtml%3B+charset%3Dutf-8'
         6        DO_FCALL                                      0
   9     7        INIT_METHOD_CALL                                         !1, 'end'
         8        INIT_FCALL                                               'rand'          //调用 内部函数 rand
         9        SEND_VAL                                                 1000          //给rand 传递参数
        10        SEND_VAL                                                 9999          //给rand 传递参数
        11        DO_ICALL                                         $3
        12        CONCAT                                           ~4      '%3Ch1%3EHello+Swoole.+%23', $3
        13        CONCAT                                           ~5      ~4, '%3C%2Fh1%3E'
        14        SEND_VAL_EX                                              ~5
        15        DO_FCALL                                      0
  10    16      > RETURN                                                   null

branch: #  0; line:     6-   10; sop:     0; eop:    16; out1:  -2
path #1: 0,
End of function %00%7Bclosure%7D%2Fhome%2Fsvn%2Fphp-traning%2Fswoole_http_server.php0x7f162b299167

//下面是 $http->on('close'  的匿名函数的opcode
Function %00%7Bclosure%7D%2Fhome%2Fsvn%2Fphp-traning%2Fswoole_http_server.php0x7f162b2991b8:
Finding entry points
Branch analysis from position: 0
Jump found. (Code = 62) Position 1 = -2
filename:       /home/svn/php-traning/swoole_http_server.php
function name:  {closure}
number of ops:  7
compiled vars:  !0 = $serv, !1 = $fd
line     #* E I O op                           fetch          ext  return  operands
-------------------------------------------------------------------------------------
  11     0  E >   RECV                                             !0
         1        RECV                                             !1
  12     2        ROPE_INIT                                     3  ~3      'Client%3A+Close'   //ROPE_INIT指令在官网没有查到，类似字符串拼接
         3        ROPE_ADD                                      1  ~3      ~3, !1
         4        ROPE_END                                      2  ~2      ~3, '.%0A'
         5        ECHO                                                     ~2
  13     6      > RETURN                                                   null

branch: #  0; line:    11-   13; sop:     0; eop:     6; out1:  -2
path #1: 0,
End of function %00%7Bclosure%7D%2Fhome%2Fsvn%2Fphp-traning%2Fswoole_http_server.php0x7f162b2991b8
```
php版本：php7.2.2
参照：http://php.net/manual/zh/internals2.opcodes.php

