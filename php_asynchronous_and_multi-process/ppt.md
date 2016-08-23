title: php 异步 与 多进程
speaker: Dray
url: https://github.com/ksky521/nodePPT
transition: cards
files: /js/demo.js,/css/demo.css

[slide]

# php 异步 与 多进程

[slide]

# 常用的实现方式 {:&.flexbox.vleft}
## Fscok
## Fork
## curl_multi_init 
## 协程
## Swoole

[slide]

# Fscok

```php
/**
 + 请求接口传输数据，不等待返回直接结束，相当于异步调用地址
 + @param  [type] $url    [description]
 + @param  array $param [description]
 + @param  string $is_wait [description] 是否等待返回
 + @return [type]         [description]
 */
public static function callUrl($url, $param = array(), $is_wait = false, $cookie = null)
{
    $res = null;
    $result = '';
    $urlinfo = parse_url($url);

    $host = $urlinfo['host'];
    $path = $urlinfo['path'];
    $query = isset($param) ? http_build_query($param) : '';

    $port = 80;
    $errno = 0;
    $errstr = '';
    $timeout = 10;

    //開啟一個 TCP/IP Socket
    $fp = fsockopen($host, $port, $errno, $errstr, $timeout);

    // 設定 header 與 body

    if (empty($query)) {
        $out = "GET " . $path . " HTTP/1.1\r\n";
    } else {
        $out = "POST " . $path . " HTTP/1.1\r\n";
    }

    $out .= "host:" . $host . "\r\n";
    $out .= "content-length:" . strlen($query) . "\r\n";
    $out .= "content-type:application/x-www-form-urlencoded\r\n";
    $out .= "connection:close\r\n";

    if (!empty($cookie)) {
        $out .= "Cookie: " . $cookie . "\r\n\r\n";
    } else {
        $out .= "\r\n";
    }

    $out .= $query;

    if (YII_ENV != 'prod') {
        \Yii::warning(PHP_EOL . $url . PHP_EOL . print_r($param, true) . PHP_EOL);
    }

    // 呼叫 WebService
    fputs($fp, $out);
    // var_dump($out);

    // 也可以选择获取server端的响应
    if ($is_wait) {
        while (!feof($fp)) {
            $result .= fgets($fp, 128);
        }
    }

    //如果不等待server端响应直接关闭socket即可
    fclose($fp);

    //处理返回值
    if (!empty($result)) {
        // split the result header from the content
        $res_arr = explode("\r\n\r\n", $result, 2);
        $content = isset($res_arr[1]) ? $res_arr[1] : '';

        //处理杂音
        $tmp = explode("\r\n", $content);
        if (
            isset($tmp[2]) &&
            hexdec($tmp[0]) &&
            $tmp[2] == 0
        ) {
            $content = $tmp[1];
        }

        $res = $content;

        //如果是 json
        if ($tmp_res = json_decode($content, true)) {
            $res = $tmp_res;
        }

        if (YII_ENV != 'prod') {
            \Yii::warning(PHP_EOL . print_r($res, true) . PHP_EOL);
        }
    }

    return $res;
}
```
[使用fscok实现异步调用PHP](http://www.laruence.com/2008/04/16/98.html)

[slide]
# Fork

```php
$process_num = 5;
print "老爸：我是老爸，我要生{$process_num}個小孩。\n";
$children = array();

for ($i = 1; $i <= $process_num; $i++) {
    $pid = pcntl_fork();
    if ($pid == -1) {
        exit(1);
    } else if ($pid) {
        /*這是老爸專區*/
        $children[] = $pid; //紀錄下每個孩子的編號
        print "老爸：生了一個第{$i}個孩子，pid是{$pid}\n";
    } else {
        /*這是小朋友區*/
        break; //直接出迴圈
    }
}

if ($pid) {
    /* 老爸會在這裡休息 */
    $status = null;
    /********************************************************
     - 下面這行的存在意義是：
     -  就算是等所有孩子先行離開以後
     -  父程序才開始等子程序
     -  父程序仍然會知道子程序已離開
     **********************************************************/
    // sleep(8);
    foreach ($children as $pid) {
        //要等每個孩子都離開才離開
        pcntl_waitpid($pid, $status);
        print "老爸：pid是{$pid}的那個孩子，回去時他告訴我他的狀況是{$status}\n";
    }
    print '老爸也要走了' . "\n";
} else {
    /*以下是小朋友遊樂區*/
    $sleep = rand(4, 10);
    print "我是第{$i}個小朋友，我要睡{$sleep}秒\n";
    sleep($sleep);
    print "我是第{$i}個小朋友，要走了\n";
    exit(0);
}
```

[PHP多进程编程初步](https://www.pureweber.com/article/php-multi-process-programming-preview/)

[slide]

# curl_multi_init 

```php
<?php
set_time_limit(0);
ob_start();
$urls = array(
    'http://www.sina.com.cn/',
    'http://www.sohu.com/',
    'http://www.163.com/',
    'http://www.zhihu.com/',
    'http://www.xunlei.com/',
    'http://www.xiachufang.com/',
    'http://www.gelonghui.com/',
    'http://www.jianshu.com/',
);

//生命一个计算脚本运行时间的类
class Timer{
    private $startTime = 0; //保存脚本开始执行时的时间（以微秒的形式保存）
    private $stopTime = 0; //保存脚本结束执行时的时间（以微秒的形式保存）

    //在脚本开始处调用获取脚本开始时间的微秒值
    function start(){
        $this->startTime = microtime(true); //将获取的时间赋值给成员属性$startTime
    }
    //脚本结束处嗲用脚本结束的时间微秒值
    function stop(){
        $this->stopTime = microtime(true); //将获取的时间赋给成员属性$stopTime
    }
    //返回同一脚本中两次获取时间的差值
    function spent(){
        //计算后4舍5入保留4位返回
        return round(($this->stopTime-$this->startTime),4);
    }
}

$timer= new Timer(); 
$timer->start(); //在脚本文件开始执行时调用这个方法

ob_end_clean();
//$save_to='./test.txt';   // 把抓取的代码写入该文件
//$st = fopen($save_to,"a");

$mh = curl_multi_init();
foreach ($urls as $i => $url) {
    $conn[$i] = curl_init($url);
    curl_setopt($conn[$i], CURLOPT_USERAGENT, "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0)");
    curl_setopt($conn[$i], CURLOPT_HEADER ,0);
    curl_setopt($conn[$i], CURLOPT_CONNECTTIMEOUT,60);
    curl_setopt($conn[$i],CURLOPT_RETURNTRANSFER,true);  // 设置不将爬取代码写到浏览器，而是转化为字符串
    echo $i;
    curl_multi_add_handle ($mh,$conn[$i]);
}

do {
    curl_multi_exec($mh,$active);
} while ($active);

foreach ($urls as $i => $url) {
    $data = curl_multi_getcontent($conn[$i]); // 获得爬取的代码字符串
    //fwrite($st,$data.$i);  // 将字符串写入文件。当然，也可以不写入文件，比如存入数据库
    file_put_contents("file/".$i.".txt",$data);
    echo $url."<br/>";
    ob_flush();
    flush();
    //usleep(200000);
} // 获得数据变量，并写入文件

foreach ($urls as $i => $url) {
    curl_multi_remove_handle($mh,$conn[$i]);
    curl_close($conn[$i]);
}

curl_multi_close($mh);
//fclose($st);
$timer->stop(); //在脚本文件结束处调用这个方法

echo "run time -> ".$timer->spent()."</b>";
?>
```
[php curl_multi_init 多线程](http://www.jianshu.com/p/ebafc55c892b)

[slide]

# 协程 - 迭代生成器

PHP 5.5 加入了对迭代生成器和协程的支持，(迭代)生成器也是一个函数,不同的是这个函数的返回值是依次返回,而不是只返回一个单独的值.或者,换句话说,生成器使你能更方便的实现了迭代器接口.
```
<?php
function xrange($start, $end, $step = 1) {
    for ($i = $start; $i <= $end; $i += $step) {
        yield $i;
    }
}

foreach (xrange(1, 1000000) as $num) {
    echo $num, "\n";
}
```

[slide]

# 协程 - 迭代生成器

上面这个xrange()函数提供了和PHP的内建函数range()一样的功能.但是不同的是range()函数返回的是一个包含值从1到100万0的数组(注：请查看手册). 而xrange()函数返回的是依次输出这些值的一个迭代器, 而不会真正以数组形式返回.

这种方法的优点是显而易见的.它可以让你在处理大数据集合的时候不用一次性的加载到内存中.甚至你可以处理无限大的数据流.

[slide]

# 协程 - 迭代生成器

```
<?php
function gen() {
    $ret = (yield 'yield1');
    var_dump($ret);
    $ret = (yield 'yield2');
    var_dump($ret);
}
 
$gen = gen();
var_dump($gen->current());    // string(6) "yield1"
var_dump($gen->send('ret1')); // string(4) "ret1"   (the first var_dump in gen)
                              // string(6) "yield2" (the var_dump of the ->send() return value)
var_dump($gen->send('ret2')); // string(4) "ret2"   (again from within gen)
                              // NULL               (the return value of ->send())
```

[slide]

# 协程 - 多任务协作

多任务协作这个术语中的“协作”很好的说明了如何进行这种切换的：它要求当前正在运行的任务自动把控制传回给调度器,这样就可以运行其他任务了. 这与“抢占”多任务相反, 抢占多任务是这样的：调度器可以中断运行了一段时间的任务, 不管它喜欢还是不喜欢. 协作多任务在Windows的早期版本(windows95)和Mac OS中有使用, 不过它们后来都切换到使用抢先多任务了. 理由相当明确：如果你依靠程序自动交出控制的话, 那么一些恶意的程序将很容易占用整个CPU, 不与其他任务共享.

现在你应当明白协程和任务调度之间的关系：yield指令提供了任务中断自身的一种方法, 然后把控制交回给任务调度器. 因此协程可以运行多个其他任务. 更进一步来说, yield还可以用来在任务和调度器之间进行通信.

[slide]

# 协程 - 多任务协作

```
<?php
include_once('Task.php');
include_once('Scheduler2.php');
include_once('SystemCall.php');

function getTaskId() {
    return new SystemCall(function(Task $task, Scheduler $scheduler) {
        $task->setSendValue($task->getTaskId());
        $scheduler->schedule($task);
    });
}

function task($max) {
    $tid = (yield getTaskId()); // <-- here's the syscall!
    for ($i = 1; $i <= $max; ++$i) {
        echo "This is task $tid iteration $i.\n";
        yield;
    }
}

$scheduler = new Scheduler;

$scheduler->newTask(task(10));
$scheduler->newTask(task(5));

$scheduler->run();
```

[slide]

# 协程 - 进阶

[在PHP中使用协程实现多任务调度](http://www.laruence.com/2015/05/28/3038.html)

[slide]

# Swoole

## 使用swoole必须要掌握的技能
1. 多线程编程
2. 进程间通信
3. 网络协议TCP/UDP的认知
4. PHP的各项基本技能

[slide]

Swoole - 比较

* Nginx：多进程Reactor
* Nginx+Lua：多进程Reactor+协程
* Golang：单线程Reactor+多线程协程
* Swoole：多线程Reactor+多进程Worker

[slide]

# Swoole - 创建Web服务器

```
$http = new swoole_http_server("0.0.0.0", 9501);

$http->on('request', function ($request, $response) {
    var_dump($request->get, $request->post);
    $response->header("Content-Type", "text/html; charset=utf-8");
    $response->end("<h1>Hello Swoole. #".rand(1000, 9999)."</h1>");
});

$http->start();
```

[slide]

# Swoole - 执行异步任务

```
$serv = new swoole_server("127.0.0.1", 9501);

//设置异步任务的工作进程数量
$serv->set(array('task_worker_num' => 4));

$serv->on('receive', function($serv, $fd, $from_id, $data) {
    //投递异步任务
    $task_id = $serv->task($data);
    echo "Dispath AsyncTask: id=$task_id\n";
});

//处理异步任务
$serv->on('task', function ($serv, $task_id, $from_id, $data) {
    echo "New AsyncTask[id=$task_id]".PHP_EOL;
    //返回任务执行的结果
    $serv->finish("$data -> OK");
});

//处理异步任务的结果
$serv->on('finish', function ($serv, $task_id, $data) {
    echo "AsyncTask[$task_id] Finish: $data".PHP_EOL;
});

$serv->start();
```


[slide]

# 相关资料

* [PHP并发IO编程之路](http://rango.swoole.com/archives/508)
* [关于C++、PHP和Swoole](http://rango.swoole.com/archives/473)
* [node.js与swoole相比优势与劣势分析](http://ju.outofmemory.cn/entry/120980)

[slide]

# PHP并发IO相关扩展

* Stream：PHP内核提供的socket封装
* Sockets：对底层Socket API的封装
* Libevent：对libevent库的封装
* Event：基于Libevent更高级的封装，提供了面向对象接口、定时器、信号处理的支持
* Pcntl/Posix：多进程、信号、进程管理的支持
* Pthread：多线程、线程管理、锁的支持
* PHP还有共享内存、信号量、消息队列的相关扩展
* PECL：PHP的扩展库，包括系统底层、数据分析、算法、驱动、科学计算、图形等都有。如果PHP标准库中没有找到，可以在PECL寻找想要的功能。

[slide]

PPT 地址 https://github.com/DrayChou/php-doc

nodeppt 是基于nodejs写的支持 **Markdown!** 语法的网页PPT，当前版本：1.4.2

Github：https://github.com/ksky521/nodePPT
