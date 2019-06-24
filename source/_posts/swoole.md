---
title: swoole学习笔记
date: 2019/6/24
toc: true
catagories:
- swoole
tags:
- swoole

---
swoole初探
<!--more-->
## TCP服务器
tcp.php
```php
<?php
$serv = new Swoole\Server("127.0.0.1",9501);

$serv->set([
    'worker_num'=>8,//worker进程数(ps aft |grep tcp.php可查看到)
    "max_request"=>10000,

]);
//监听连接事件
$serv->on('connect',function($serv, $fd, $reactor_id){
    echo "client :{$reactor_id}-{$fd}-connect.\n";
});
//监听数据接收事件
$serv->on("receive",function($serv, $fd, $from_id, $data){
    $serv->send($fd,"Server: {$from_id}-{$fd}-{$data}");
});
//监听连接关闭事件
$serv->on("close",function($serv,$fd){
    echo "client: Close.\n";
});
//启动服务器
$serv->start();
```
终端测试：
```shell
#开启
php tcp.php
#另开一个终端连接测试(可发送数据)
telnet 127.0.0.1 9501
```

## TCP客户端
tcp_client.php(替代上一步telnet)
```php
<?php
$client = new Swoole\client(SWOOLE_SOCK_TCP);
if(!$client->connect('127.0.0.1',9501)){
    echo '连接失败';
    exit();
}
//php cli常量
fwrite(STDOUT,"请输入消息：");
$msg = trim(fgets(STDIN));
//发送消息给tcp server
$client->send($msg);
//接收来自server的数据
$result = $client->recv();
echo $result;
```
终端测试：
```shell
#先打开tcp服务器
php tcp.php
#另开一个终端打开tcp客户端连接测试
php tcp_client.php
```

## UDP服务器
UDP服务器与TCP服务器不同，UDP没有连接的概念。启动Server后，客户端无需Connect，直接可以向Server监听的9502端口发送数据包。
```php
udp.php
<?php
$serv = new Swoole\server("127.0.0.1",9502,SWOOLE_PROCESS,SWOOLE_SOCK_UDP);

$serv->on('Packet', function ($serv, $data, $clientInfo) {
    $serv->sendto($clientInfo['address'], $clientInfo['port'], "Server ".$data);
    var_dump($clientInfo);
});

$serv->start();
```
终端测试：
```shell
#启动udp服务器
php udp.php
#用nc命令进行连接测试(安装：yum install -y nc)
nc -u 127.0.0.1 9502
hello
Server hello
```

## UDP客户端
udp_client.php
```php
<?php
$client = new Swoole\client(SWOOLE_SOCK_UDP);

fwrite(STDOUT,"请输入消息：");
$msg = trim(fgets(STDIN));

$client->sendto('127.0.0.1',9502,$msg);
$result = $client->recv();
echo $result;
```
终端测试：
```shell
#启动udp服务器
php udp.php
#打开另一终端连接测试
php udp_client.php
```

## HTTP服务器
与php内置服务器差不到的原理，不走php-fpm
ws_server.php
```php
<?php
$http = new Swoole\Http\Server("0.0.0.0",8811);
//可设置静态文件
//底层收到Http请求会先判断document_root路径下是否存在此文件，如果存在会直接发送文件内容给客户端，不再触发onRequest回调
$http->set(
    [
        'enable_static_handler'=>true,
        'document_root'=>"/vagrant_data/study/swoole/data",//静态文件根目录
    ]
);
$http->on("request",function($request, $response){
    //获取请求的GET参数
    print_r($request->get);
    //可设置cookie，与PHP的setcookie一样
    $response->cookie('name','cynickimi',time()+1800);
    //向客户端浏览器发送HTML内容，string
    $response->end('GET:'.json_encode($request->get));
});

$http->start();
```

## WebSocket服务器
. 服务端
- WebSocket协议是基于TCP的一种新的网络协议。它实现了浏览器与服务器全双工通信---允许服务器主动发送信息给客户端。
- 协议标识符`ws`  `wss`

```php
<?php
$server = new Swoole\WebSocket\Server("0.0.0.0", 8812);
//设置静态文件来加载客户端
$server->set(
    [
        'enable_static_handler'=>true,
        'document_root'=>"/vagrant_data/study/swoole/data",
    ]
);
//监听websocket连接打开事件
$server->on("open",'onOpen');
function onOpen($server, $request){
    print_r($request->fd);
}
//监听ws消息事件
$server->on('message',function(Swoole\WebSocket\Server $server,$frame){
    echo "receive from client {$frame->fd}:{$frame->data},opcode:{$frame->opcode},fin:{$frame->finish}\n";
    $server->push($frame->fd,"this is server receive:push success!");
});

$server->on('close',function($ser, $fd){
    echo "client {$fd} closed\n";
});

$server->start();
```
.客户端
ws_client.html(位置：websocket设置的document_root)
```javascript
<script>
    var wsUrl = "ws://192.168.1.10:8812";
    var websocket = new WebSocket(wsUrl);

    websocket.onopen = function(evt){
        websocket.send("hello,i am client.");
        console.log("connect-swoole-success.");
    }

    websocket.onmessage = function (evt) {
        console.log('ws-server-return-data:'+evt.data);
    }

    websocket.onclose = function(evt){
        console.log("close");
    }

    websocket.onerror = function (evt, e){
        console.log("error:" + evt.data);
    }
</script>
```
.运行测试
```shell
#启动websocket服务端
php ws_server.php
#浏览器访问客户端
http://192.168.1.10：8812/ws_client.html
```

## Task任务（异步）
ws.php(面向对象风格)
```php
<?php
class Ws{
    CONST HOST = "0.0.0.0";
    CONST PORT = 8812;

    public  $ws = null;
    public function __construct()
    {
        $this->ws = new Swoole\WebSocket\Server("0.0.0.0", 8812);

        $this->ws->set([
            'worker_num'=>2,
            "task_worker_num"=>2,//设置异步任务的工作进程数量
        ]);
        $this->ws->on("open",[$this,"onOpen"]);
        $this->ws->on("message",[$this,"onMessage"]);
        $this->ws->on("task",[$this,"onTask"]);
        $this->ws->on("finish",[$this,"onFinish"]);
        $this->ws->on("close",[$this,"onClose"]);

        $this->ws->start();
    }

    public function onOpen($ws,$request)
    {
        var_dump($request->fd);
    }

    public function onMessage($ws,$frame)
    {
        echo "ser-push-message:{$frame->data}\n";
        $data = [
            'task'=>1,
            "fd"=>$frame->fd,
        ];
        $ws->task($data);//出发onTask,异步
        $ws->push($frame->fd,"server-push:".date("Y-m-d H:i:s"));
    }

    public function onTask($serv,$taskId,$workId,$data)
    {
        print_r($data);
        //耗时场景
        sleep(10);
        return "on task finish.";//告诉worker
    }

    /**
     * @param $serv
     * @param $taskId
     * @param $data onTask函数return的内容
     */
    public function onFinish($serv, $taskId, $data)
    {
        echo "taskID:{$taskId}\n";
        echo "finish-data-success:{$data}.\n";
    }

    public function onClose($ws,$fd)
    {
        echo "clientid:{$fd}";
    }
}

$obj = new Ws();
```
测试：
```shell
#websocket服务端
php ws.php
#启动websocket客户端
php http_server.php
http://192.168.1.10:8811/ws_client.php
```

## 异步文件系统IO
**4.3.0之后的版本用协程代替(coroutine)**
### 读取文件
```php
<?php
$result = Swoole\Async::readfile(__DIR__."/1.txt",function($filename,$fileContent){
    echo "filename:".$filename.PHP_EOL;
    echo "content:".$fileContent.PHP_EOL;
});
var_dump($result);//优先输出
echo "start".PHP_EOL;//优先输出
```

### 写入文件
```php
<?php
$content = date("Y-m-d H:i:s");
swoole_async_writefile(__DIR__."/1.log",$content,function($filename){
    echo "success".PHP_EOL;
},FILE_APPEND);

echo "start".PHP_EOL;
```

## 异步MySQL
**Swoole\MySQL已内置到Swoole中，无需通过--enable-async-mysql编译参数开启**

```php
<?php
class AysMysql{
    public $dbSource = "";
    public $dbConfig = "";
    public function __construct()
    {
        $this->dbSource = new Swoole\Mysql;
        $this->dbConfig = [
            'host' => '127.0.0.1',
            'port' => 3306,
            'user' => 'root',
            'password' => 'root',
            'database' => 'swoole',
            'charset' => 'utf8',
        ];
    }

    public function show_tables()
    {
        $this->dbSource->connect($this->dbConfig, function($db, $result){
           echo "connect.".PHP_EOL;
           if($result === false){
               var_dump($db->connect_error);
           }

           $sql = 'show tables';
           $db->query($sql, function(Swoole\MySQL $db, $result){
               if ($result === false)
               {
                   var_dump($db->error, $db->errno);
               }
               elseif ($result === true )
               {
                   var_dump($db->affected_rows, $db->insert_id);
               }
               var_dump($result);
               $db->close();
           });
        });
        return true;
    }
}

$obj = new AysMysql();
$obj->show_tables();
echo "start".PHP_EOL;//优先输出
```
## 进程
process.php
```php
<?php
$process = new swoole_process(function(swoole_process $pro){
    $pro->exec("/usr/local/php7.2/bin/php",[__DIR__."/../server/http_server.php"]);
}, false);
$pid = $process->start();//创建子进程
echo $pid.PHP_EOL;//子进程pid
swoole_process::wait();//回收结束运行的子进程。
```
测试：
运行process.php,会打印出子进程id,即是创建的http_server进程的id
![](https://github.com/CynicKimi/images/raw/master/process1.png)
查看process.php的进程号
![](https://github.com/CynicKimi/images/raw/master/process2.png)
查看process.php(4124)进程下的所有进程关系：
![](https://github.com/CynicKimi/images/raw/master/process3.png)
查看http_server.php的进程关系：
![](https://github.com/CynicKimi/images/raw/master/process4.png)

使用场景：模拟读取4个url，每个耗时1s，一般顺序执行需要4s
```php
<?php
echo "process-start-time:".date("Ymd H:i:s").PHP_EOL;
$workers = [];
$curl = [
  'https://www.baidu.com',
  'https://www.taobao.com',
  'https://www.php.net/',
  'https://wiki.swoole.com',
];

function curlData($url){
    //模拟读取耗时场景
    sleep(1);
    return 'read'.$url." success.".PHP_EOL;
}

for($i = 0; $i<4; $i++){
    $process = new Swoole\Process(function(Swoole\Process $worker) use($i,$curl){
        $content = curlData($curl[$i]);
        echo $content.PHP_EOL;
    },true);
    $pid = $process->start();
    $workers[$pid] = $process;
}

foreach($workers as $process){
    echo $process->read();//从管道中读取数据。
}

echo "process-end-time:".date("Ymd H:i:s").PHP_EOL;
```
执行结果：1s就执行结束
![](https://github.com/CynicKimi/images/raw/master/process_test.png)

## 内存表
使用场景：可用于数据共享。觉得有点类似redis
```php
<?php
//创建内存表
$table = new swoole_table(1024);
$table->column('id', $table::TYPE_INT, 4);
$table->column('name',$table::TYPE_STRING, 64);
$table->column('age', $table::TYPE_INT, 3);
$table->create();
//新增内容
$table->set('cynickimi',['id'=>1,'name'=>'cynickimi','age'=>24]);
//另一种新增内容方法
$table['cynickimi2']=[
    'id'=>2,
    'name'=>'cynickimi2',
    'age'=>24,
];
//两种获取数据的方法
print_r($table->get('cynickimi'));
print_r($table['cynickimi']);
//自增
$table->incr('cynickimi2','age',2);
print_r($table['cynickimi2']);

//执行完释放内存，可用于共享数据
```


## 协程

### 协程MySQL
```php
<?php
$http = new Swoole\Http\Server('0.0.0.0', 8811);
$http->on('request',function($request, $response){
    $swoole_mysql = new Swoole\Coroutine\MySQL();

    $swoole_mysql->connect([
        'host'=>'127.0.0.1',
        'port'=>3306,
        'user'=>'root',
        'password'=>'root',
        'database'=>'swoole',
    ]);
    $res = $swoole_mysql->query("show tables");
    print_r($res);
});

$http->start();
```
测试：
```shell
php mysql.php
curl 127.0.0.1:8811
```

### 协程Redis
```php
<?php
$http = new swoole_http_server('0.0.0.0', 8811);
$http->on('request',function($request, $response){
    $redis = new Swoole\Coroutine\Redis();
    $redis->connect('127.0.0.1',6379);
    $value = $redis->get($request->get['key']);
    $response->header("Content-Type","text/plain");
    $response->end($value);
});

$http->start();
```
测试：
```shell
php redis.php
#获取redis中key为name的值
curl 127.0.0.1:8811?key=name
```



