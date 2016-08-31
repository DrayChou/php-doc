# PHP7新增功能详解

`php`

---

这一篇主要是来详细分析php7.0的新增功能。

## 一、性能与底层

### PHP7速度是 PHP5.6 的两倍

php7 最显著的变化就是性能的极大提升，已接近Facebook开发的PHP执行引擎HHVM。在WordPress基准性能测试中，速度比5.6版本要快2~3倍，大大减少了内存占用。PHP7在语言上也有一些变化，比如添加返回类型声明、增加了一些新的保留关键字等。在安全方面，去除了PHP安全模式，添加魔术引号等。不仅如此，新版还支持64位，而且包含最新版Zend引擎。

### 测试一下

1. 很简单的一个例子，生成一个 60 万元素的数组，通过查找key 的方式，来确定key是否存在。


    1. <?php
    2. $a =[];
    3. for($i=0;$i<600000;$i++){
    4.   $a[$i]= $i;
    5. }
    6. foreach($a as $item){
    7.  array_key_exists($item, $a);
    8. }



我们分别在`php5.6.11`和`php7.0.4`来测试下性能。

php5.6.11


    1. ➜ time php 1.php
    2. 0.67s user 0.06s system 67% cpu 1.078 total
    3. ➜ time php 1.php
    4. 0.68s user 0.06s system 98% cpu 0.748 total
    5. ➜ time php 1.php
    6. 0.65s user 0.06s system 67% cpu 1.052 total



三次平均下来，大致是 user使用 0.65秒，system使用0.06秒，67%的cpu率。总共1秒左右。

再看php7的情况


    1. ➜  time /usr/local/opt/php70/bin/php 1.php
    2. 0.52s user 0.02s system 98% cpu 0.544 total
    3. ➜  time /usr/local/opt/php70/bin/php 1.php
    4. 0.49s user 0.02s system 99% cpu 0.513 total
    5. ➜  time /usr/local/opt/php70/bin/php 1.php
    6. 0.51s user 0.02s system 98% cpu 0.534 total



对比下来，user使用时间下降20%左右，system使用时间下降70%，cpu使用率更高高达98%。总体时间减少为。0.5秒。

这个例子看下来，效率提供了2倍。确实不错。

1. 再看一个例子。同样也是生成一个 60 万元素的数组，查找 value是否存在。


    1. <?php
    2. $a =[];
    3. for($i=0;$i<600000;$i++){
    4.     $a[$i]= $i;
    5. }
    6. foreach($a as $i){
    7.     array_search($i, $a);
    8. }
    9. ?>



先看php5.6.11


    1. ➜  testPHP time php 2.php
    2. 0.68s user 0.03s system 66% cpu 1.077 total
    3. ➜  testPHP time php 2.php
    4. 0.68s user 0.02s system 98% cpu 0.710 total
    5. ➜  testPHP time php 2.php
    6. 0.68s user 0.02s system 98% cpu 0.713 total
    7. ➜  testPHP time php 2.php
    8. 0.69s user 0.02s system 98% cpu 0.721 total



再接着看php7.0.4


    1. ➜  testPHP time /usr/local/opt/php70/bin/php 2.php
    2. 0.12s user 0.02s system 69% cpu 0.201 total
    3. ➜  testPHP time /usr/local/opt/php70/bin/php 2.php
    4. 0.11s user 0.01s system 97% cpu 0.131 total
    5. ➜  testPHP time /usr/local/opt/php70/bin/php 2.php
    6. 0.11s user 0.01s system 96% cpu 0.130 total



明显看出，快了6倍多。厉害。

## 二、新特性

### 1. 更多的标量类型声明

现在php的标量有两种模式: `强制` (默认) 和`严格模式`。 现在可以使用下列类型参数（无论用强制模式还是严格模式）： 字符串(string), 整数 (int), 浮点数 (float), 以及布尔值 (bool)。它们扩充了PHP5中引入的其他类型：类名，接口，数组和 回调类型。在旧版中，函数的参数申明只能是(Array $arr)、(CLassName $obj)等，基本类型比如Int，String等是不能够被申明的。

怎么理解呢？php7之前的版本，我们要想限定一个函数的参数的类型，只有`array`或者`class`2种。

php7之前：


    1. classMyInfo
    2. {
    3. public $a =123;
    4. publicfunction getInfo(array $a, $b)
    5. {
    6.         var_dump($a, $b);
    7. }
    8. }
    9. function getClass(MyInfo $a){
    10.     var_dump($a->a);
    11. }



我们想限定 `getInfo`的第一个参数，必须是数组，所以，我们可以在参数`$a`前加一个`array`。来申明。

同样，我们想`getClass`的参数，必须是一个类，所以我们就用这个类的`className`前坠来申明，起到强制使用的目的。

**php7之前，只有这2种标量可以使用。**

我们来使用一下：


    1. $info =newMyInfo();
    2. $info->getInfo([1,2,3,4],4);



我们按照规定的来，第一个参数，传数组，结果当然是正常打印：


    1. ➜  testPHP php 3.php
    2. array(3){
    3. [0]=>
    4. int(1)
    5. [1]=>
    6. int(2)
    7. [2]=>
    8. int(3)
    9. }
    10. int(4)



要是我们不安装规定来，就会报知名错误：


    1. $info =newMyInfo();
    2. $info->getInfo(122,0);



报错：


    1. PHP Catchable fatal error:Argument1 passed to MyInfo::getInfo() must be of the type array, integer given, called in/Users/yangyi/www/testPHP/3.php on line 25anddefinedin/Users/yangyi/www/testPHP/3.php on line 8
    2. PHP Stack trace:
    3. PHP   1.{main}()/Users/yangyi/www/testPHP/3.php:0
    4. PHP   2.MyInfo->getInfo()/Users/yangyi/www/testPHP/3.php:25



使用类也一样：


    1. $info =newMyInfo();
    2. getClass($info);



输出结果：


    1. ➜  testPHP php 3.php
    2. int(123)



同样，我们传入别的参数，就会报错：


    1. getClass(123);




    1. ➜  testPHP php 3.php
    2. PHP Catchable fatal error:Argument1 passed to getClass() must be an instance of MyInfo, integer given, called in/Users/yangyi/www/testPHP/3.php on line 27anddefinedin/Users/yangyi/www/testPHP/3.php on line 17
    3. PHP Stack trace:
    4. PHP   1.{main}()/Users/yangyi/www/testPHP/3.php:0
    5. PHP   2. getClass()/Users/yangyi/www/testPHP/3.php:27



我们回到这次php7的升级，它扩充了标量的类型，增加了`bool`、`int`、`string`、`float`。

**php7有2种两种模式: `强制` (默认) 和`严格模式`。**

#### 强制模式

强制模式是默认模式，强制模式下，它会帮我们把数字类型的string类型，int整型，bool，强制类型。其他类型不能转换，就会报错。

还是刚才的例子：


    1. classMyInfo
    2. {
    3. public $a =123;
    4. publicfunction get1(bool $b)
    5. {
    6.         var_dump($b);
    7. }
    8. publicfunction get2(int $b)
    9. {
    10.         var_dump($b);
    11. }
    12. publicfunction get3(string $b)
    13. {
    14.         var_dump($b);
    15. }
    16. publicfunction get4(float $b)
    17. {
    18.         var_dump($b);
    19. }
    20. publicfunction get5(array $b)
    21. {
    22.         var_dump($b);
    23. }
    24. }



我们先全部传入`int 1`


    1. $info =newMyInfo();
    2. $info->get1(1);
    3. $info->get2(1);
    4. $info->get3(1);
    5. $info->get4(1);



看打印结果，它已经帮我们强制转换了。


    1. ➜  testPHP /usr/local/opt/php70/bin/php 3.php
    2. /Users/yangyi/www/testPHP/3.php:11:
    3. bool(true)
    4. /Users/yangyi/www/testPHP/3.php:19:
    5. int(1)
    6. /Users/yangyi/www/testPHP/3.php:26:
    7. string(1)"1"
    8. /Users/yangyi/www/testPHP/3.php:33:
    9. double(1)



我们继续，传入 `string 1.23` :


    1. $info =newMyInfo();
    2. $info->get1('1.23');
    3. $info->get2('1.23');
    4. $info->get3('1.23');
    5. $info->get4('1.23');



看下，打印结果。也已经帮我们强制转换了。


    1. ➜  testPHP /usr/local/opt/php70/bin/php 3.php
    2. /Users/yangyi/www/testPHP/3.php:11:
    3. bool(true)
    4. /Users/yangyi/www/testPHP/3.php:19:
    5. int(1)
    6. /Users/yangyi/www/testPHP/3.php:26:
    7. string(4)"1.23"
    8. /Users/yangyi/www/testPHP/3.php:33:
    9. double(1.23)



但是我们如果参数是`array`就没法强制转换，就会报错了：


    1. $info->get5('1.23');




    1.  testPHP /usr/local/opt/php70/bin/php 3.php
    2. PHP Fatal error:UncaughtTypeError:Argument1 passed to MyInfo::get5() must be of the type array,string given, called in/Users/yangyi/www/testPHP/3.php on line 54anddefinedin/Users/yangyi/www/testPHP/3.php:37



我们在PHP5.6.11运行这些代码会报错吗？试一试：


    1. $info =newMyInfo();
    2. $info->get1('1.23');
    3. $info->get2('1.23');
    4. $info->get3('1.23');
    5. $info->get4('1.23');




    1. ➜  testPHP php 3.php
    2. PHP Catchable fatal error:Argument1 passed to MyInfo::get1() must be an instance of bool,string given, called in/Users/yangyi/www/testPHP/3.php on line 48anddefinedin/Users/yangyi/www/testPHP/3.php on line 8



好吧。直接报错了，虽然错误提示也是说类型错误，但是，其他是不支持这些类型的申明。

#### 严格模式

前面说了，强制模式下，它会帮我们强制转换，那么严格模式下呢？

首先，如何打开`严格模式`呢？


    1. <?php
    2. declare(strict_types=1);



加上就可以了，这样，就进入严格模式，参数必须符合规定，不然报错：

我们加上这句话，再运行下：


    1. <?php
    2. declare(strict_types=1);
    3. ...
    4. ...
    5. $info =newMyInfo();
    6. $info->get1('1.23');
    7. $info->get2('1.23');
    8. $info->get3('1.23');
    9. $info->get4('1.23');



运行，看下结果，果然直接报错了。


    1. PHP Fatal error:UncaughtTypeError:Argument1 passed to MyInfo::get1() must be of the type boolean,string given, called in/Users/yangyi/www/testPHP/3.php on line 49anddefinedin/Users/yangyi/www/testPHP/3.php:9



### 2. 返回值类型声明

我们知道php的函数是没有返回值类型的，`return`啥类型，就是啥类型。php7中增加了返回值类型，我们可以定义一个函数的返回值类型。

和php7升级的标量类型声明一样，return的类型可以是以下这些：`bool`、`int`、`string`、`float`、`array`、`class`。

举个例子来说，我们希望一个函数的返回值是一个数组，我们可以这样子书写：

    ：array {}  // 冒号＋返回类型



    1. function returnInfo ($a): array {
    2. return $a;
    3. }
    4. var_dump(returnInfo([1,2,3]));



是不是觉得很奇怪，又无可思议！！！

查看打印结果：


    1. ➜  testPHP /usr/local/opt/php70/bin/php 3.php
    2. /Users/yangyi/www/testPHP/3.php:64:
    3. array(3){
    4. [0]=>
    5. int(1)
    6. [1]=>
    7. int(2)
    8. [2]=>
    9. int(3)
    10. }



同样，我们想返回是`int`整型：


    1. function returnInfo ($a):int{
    2. return $a;
    3. }
    4. var_dump(returnInfo('1.233'));



查看结果，他已经帮我们强制转换成整型了。


    1. ➜  testPHP /usr/local/opt/php70/bin/php 3.php
    2. /Users/yangyi/www/testPHP/3.php:64:
    3. int(1)



同样，我们可以返回一个`class`类型的：


    1. publicfunction getLogger():Logger{
    2. return $this->logger;
    3. }



默认，也是强制模式，会帮我们转换，如果，我们想使用严格模式，同样是一样的，在文件头部加上：


    1. <?php
    2. declare(strict_types=1);



就可以了，这样，我们规定返回值是什么类型，就必须得是这样，不然就报致命报错。

### 3. null合并运算符 (??)

由于日常使用中存在大量同时使用三元表达式和 isset()的情况， php7增加了一个新的语法糖 : null合并运算符 (??) 

如果变量存在且值不为NULL, 它就会返回自身的值，否则返回它的第二个操作数。


    1. //php version = 7 
    2. $username = $user ??'nobody';
    3. //php  version < 7 得这样使用：
    4. $username = isset($_GET['user'])? $_GET['user']:'nobody';



确实方便了很多。

我记得php5.3的更新中，加入了 三元运算符简写形式：

    $a ?: $b


千万别和`??`搞混淆了！！！

`$a ?: $b`的意思是 $a为`true`时，直接返回`$a`, 否则返回`$b`

`$a ?? $b`的意思是 $a `isset($a)`为`true`, 且不为NULL, 就返回`$a`, 否则返回`$b`。

看例子：


    1. $user =0;
    2. $username = $user ??'nobody';
    3. echo $username;//输出 0，因为 0 存在 且 不为NULL。
    4. $username = $user ?:'nobody';
    5. echo $username;//输出 'nobody'，因为 0 为 false



### 4. 太空船操作符（组合比较符)

php7 中，新加入了一个比较符号：`<=>` ,因为长相像太空船，所以，也叫太空船操作符。

它有啥用呢？

<=>用于比较两个表达式。当`$a`小于、等于或大于`$b`时它分别返回`-1`、`0`或`1`。

看例子：


    1. <?php
    2. // Integers
    3. echo 1<=>1;// 0
    4. echo 1<=>2;// -1
    5. echo 2<=>1;// 1
    6. // Floats
    7. echo 1.5<=>1.5;// 0
    8. echo 1.5<=>2.5;// -1
    9. echo 2.5<=>1.5;// 1
    10. // Strings
    11. echo "a"<=>"a";// 0
    12. echo "a"<=>"b";// -1
    13. echo "b"<=>"a";// 1
    14. ?>



其实，蛮多地方可以派上用场的。

### 5. 通过define()定义常量数组

`Array`类型的常量现在可以通过 `define()`来定义。在 PHP5.6 中仅能通过`const`定义。

在php5.3中，增加了可以使用`const`来申明常量，替代`define()`函数，但是只能申明一些简单的变量。


    1. //旧式风格：
    2. define("XOOO","Value");
    3. //新式风格：
    4. const XXOO ="Value";
    5. //const 形式仅适用于常量，不适用于运行时才能求值的表达式：
    6. // 正确
    7. const XXOO =1234;
    8. // 错误
    9. const XXOO =2*617;



在php5.6中，又对const进行来升级，可以支持上面的运算了。


    1. const A =2;
    2. const B = A +1;



但是，一只都是在优化const，可是确把`define()`给搞忘记了，php 5.6申明一个数组常量，只能用const。所以，在 php7 中把 `define()`申明一个数组也给加上去了。


    1. //php 7
    2. define ('AWS',[12,33,44,55]);
    3. // php < 7
    4. const QWE =[12,33,44,55];
    5. echo AWS[1];//12
    6. echo QWE[2];//33



至此，到php7版本，`define()`的功能和`const`就一摸一样了，所以，你随便用哪一个都可以，但是因为在`class`类中，什么常量是`const`。所以，我们就统一用`const`申明常量好了。

### 6. 匿名类

现在已经支持通过`new class` 来实例化一个匿名类，这可以用来替代一些`用后即焚`的完整类定义。

看下这个官方文档上的一个栗子：


    1. <?php
    2. interfaceLogger{
    3. publicfunction log(string $msg);
    4. }
    5. classApplication{
    6. private $logger;
    7. publicfunction getLogger():Logger{
    8. return $this->logger;
    9. }
    10. publicfunction setLogger(Logger $logger){
    11.          $this->logger = $logger;
    12. }
    13. }
    14. $app =newApplication;
    15. $app->setLogger(newclassimplementsLogger{
    16. publicfunction log(string $msg){
    17.         echo $msg;
    18. }
    19. });
    20. var_dump($app->getLogger());
    21. ?>



我们先输出的打印的结果，显示为匿名类：


    1. classclass@anonymous#2 (0) {
    2. }



我们来分解下，还原被偷懒的少写的代码：


    1. class logClass implementsLogger{
    2. publicfunction log(string $msg){
    3.         echo $msg;
    4. }
    5. }
    6. $app =newApplication;
    7. $log2 =new logClass;
    8. $app->setLogger($log2);



输出结果为：


    1. class logClass#2 (0) {
    2. }



虽然代码简洁了很多，但是还是有点不适应，多用用就好了。

还记得php中的匿名函数嘛？在php5.3中新增的匿名函数，结合新的，顺便复习下：


    1. function arraysSum(array ...$arrays): array {
    2. return array_map(function(array $array):int{
    3. return array_sum($array);
    4. }, $arrays);
    5. }
    6. print_r(arraysSum([1,2,3],[4,5,6],[7,8,9]));



输出结果为：


    1. Array
    2. (
    3. [0]=>6
    4. [1]=>15
    5. [2]=>24
    6. )



### 7. Unicode codepoint 转译语法

ps : 由于用的少，我就直接抄官网的说明了。😊

这接受一个以16进制形式的 Unicode codepoint，并打印出一个双引号或heredoc包围的 UTF-8 编码格式的字符串。 可以接受任何有效的 codepoint，并且开头的 0 是可以省略的。


    1. echo "\u{0000aa}";
    2. echo "\u{aa}";//省略了开头的0
    3. echo "\u{9999}";



看下输出：

    ª ª 香


我们在php5.6环境下执行下呢？会怎样：

    \u{aa} \u{0000aa} \u{9999}


好吧，直接原样输出了。

### 8. Closure::call() 闭包

ps : 由于用的少，我就直接抄官网的说明了。😊

Closure::call() 现在有着更好的性能，简短干练的暂时绑定一个方法到对象上闭包并调用它。


    1. <?php
    2. class A {private $x =1;}
    3. // php 7之前：
    4. $getXCB =function(){return $this->x;};
    5. $getX = $getXCB->bindTo(new A,'A');// intermediate closure
    6. echo $getX();
    7. // PHP 7：
    8. $getX =function(){return $this->x;};
    9. echo $getX->call(new A);



会输出：

    1
    1


### 9. 为unserialize()提供过滤

`unserialize` 这个函数应该不陌生，它是php中用解开用`serialize`序列化的变量。

看个栗子：


    1. <?php
    2. $a =[1,2,3,4,5,6];
    3. $b = serialize($a);
    4. $c = unserialize($b);
    5. var_dump($a, $b, $c);



打印结果为：


    1. array(6){
    2. [0]=>
    3. int(1)
    4. [1]=>
    5. int(2)
    6. [2]=>
    7. int(3)
    8. [3]=>
    9. int(4)
    10. [4]=>
    11. int(5)
    12. [5]=>
    13. int(6)
    14. }
    15. string(54)"a:6:{i:0;i:1;i:1;i:2;i:2;i:3;i:3;i:4;i:4;i:5;i:5;i:6;}"
    16. array(6){
    17. [0]=>
    18. int(1)
    19. [1]=>
    20. int(2)
    21. [2]=>
    22. int(3)
    23. [3]=>
    24. int(4)
    25. [4]=>
    26. int(5)
    27. [5]=>
    28. int(6)
    29. }



现在php7中`unserialize`会变得更佳好用，它多了一个参数，用来反序列化包涵`class`的过滤不需要的类，变的更加安全。


    1.     unserialize($one,["allowed_classes"=>true]);
    2.     unserialize($one,["allowed_classes"=>false]);
    3.     unserialize($one,["allowed_classes"=>[class1,class2,class3]]);



举个例子，先序列化一个类。


    1. classMyInfo{
    2. publicfunction getMyName()
    3. {
    4. return'phper';
    5. }
    6. }
    7. $phper =newMyInfo();
    8. $one = serialize($phper);
    9. //参数allowed_classes 设置为 true，表示允许解析class
    10. $two = unserialize($one,["allowed_classes"=>true]);
    11. //参数allowed_classes 设置为 false，表示不允许解析class
    12. $three = unserialize($one,["allowed_classes"=>false]);
    13. //不加参数。正常解析。
    14. $four = unserialize($one);
    15. //只允许解析 类 MyInfo1。
    16. $five = unserialize($one,["allowed_classes"=>["MyInfo1"]]);
    17. //分别输出下 getMyName方法;
    18. var_dump($one);
    19. var_dump($two->getMyName());
    20. var_dump($three->getMyName());
    21. var_dump($four->getMyName());
    22. var_dump($five->getMyName());



发现3和5直接报致命错误了：


    1. PHP Fatal error:  main():The script tried to execute a method or access a property of an incomplete object.Pleaseensure that the class definition "MyInfo" of the object you are trying to operate on was loaded _before_ unserialize() gets called or provide a __autoload()function to load the class definition  in/Users/yangyi/www/php7/5.php on line 22



大致意思就是，没权限解析。

所以，我闷改一下：


    1. $three = unserialize($one,["allowed_classes"=>true]);
    2. $five = unserialize($one,["allowed_classes"=>["MyInfo"]]);



再输出，就正常了。


    1. /Users/yangyi/www/php7/5.php:22:
    2. string(17)"O:6:"MyInfo":0:{}"
    3. /Users/yangyi/www/php7/5.php:23:
    4. string(5)"phper"
    5. /Users/yangyi/www/php7/5.php:24:
    6. string(5)"phper"
    7. /Users/yangyi/www/php7/5.php:25:
    8. string(5)"phper"
    9. /Users/yangyi/www/php7/5.php:26:
    10. string(5)"phper"



发现我目前为止并没用到，并没有什么乱用，好吧，继续下一个。

### 10. IntlChar

ps : 由于用的少，我就直接抄官网的说明了。😊

新增加的 [IntlChar](http://php.net/manual/zh/class.intlchar.php) 类旨在暴露出更多的 ICU 功能。这个类自身定义了许多静态方法用于操作多字符集的 unicode 字符。


    1. <?php
    2. printf('%x',IntlChar::CODEPOINT_MAX);
    3. echo IntlChar::charName('@');
    4. var_dump(IntlChar::ispunct('!'));



以上例程会输出：


    1. 10ffff
    2. COMMERCIAL AT
    3. bool(true)



若要使用此类，请先安装 [Intl](http://php.net/manual/zh/book.intl.php) 扩展。

### 11. Expectations (预期)

ps : 由于用的少，我就直接抄官网的说明了。😊

预期是向后兼用并增强之前的 assert() 的方法。 它使得在生产环境中启用断言为零成本，并且提供当断言失败时抛出特定异常的能力。

老版本的API出于兼容目的将继续被维护，assert()现在是一个语言结构，它允许第一个参数是一个表达式，而不仅仅是一个待计算的 string或一个待测试的boolean。


    1. <?php
    2. ini_set('assert.exception',1);
    3. classCustomErrorextendsAssertionError{}
    4. assert(false,newCustomError('Some error message'));
    5. ?>



输出：


    1. Fatal error:UncaughtCustomError:Some error message



### 12. Group use declarations

从同一 `namespace` 导入的类、函数和常量现在可以通过单个 use 语句一次性导入了。不熟悉命名空间的可以看我写的这篇文章:  [PHP中的命名空间](https://www.zybuluo.com/phper/note/65479)


    1. <?php
    2. //PHP7之前
    3. use some\namespace\ClassA;
    4. use some\namespace\ClassB;
    5. use some\namespace\ClassC as C;
    6. usefunction some\namespace\fn_a;
    7. usefunction some\namespace\fn_b;
    8. usefunction some\namespace\fn_c;
    9. useconst some\namespace\ConstA;
    10. useconst some\namespace\ConstB;
    11. useconst some\namespace\ConstC;
    12. // PHP7之后
    13. use some\namespace\{ClassA,ClassB,ClassCas C};
    14. usefunction some\namespace\{fn_a, fn_b, fn_c};
    15. useconst some\namespace\{ConstA,ConstB,ConstC};
    16. ?>



### 13. (Generator Return Expressions )生成器的返回值

在PHP5.5引入生成器的概念。生成器函数每执行一次就得到一个yield标识的值。在PHP7中，当生成器迭代完成后，可以获取该生成器函数的返回值。通过Generator::getReturn()得到。

如果对生成器不是很懂的，可以看我写的这片文章 [php中生成器(generator, yield) 的用法](https://www.zybuluo.com/phper/note/431945)

看个例子：


    1. # 闭包匿名函数
    2. $gen =(function(){
    3. yield1;
    4. yield2;
    5. return3;
    6. })();
    7. foreach($gen as $val){
    8.     echo $val, PHP_EOL;
    9. }
    10. #必须循环完毕
    11. echo $gen->getReturn(), PHP_EOL;



输出：


    1. 1
    2. 2
    3. 3#return



### 14. (Generator delegation)生成器中引入其他生成器

在生成器中可以引入另一个或几个生成器，只需要写`yield from functionName1`

看个栗子：


    1. <?php
    2. function gen()
    3. {
    4. yield1;
    5. yield2;
    6. yieldfrom gen2();
    7. }
    8. function gen2()
    9. {
    10. yield3;
    11. yield4;
    12. }
    13. foreach(gen()as $val)
    14. {
    15.     echo $val, PHP_EOL;
    16. }
    17. ?>



会输出：


    1. 1
    2. 2
    3. 3
    4. 4



### 15. (Integer division with intdiv() )整型相除函数intdiv

接收两个参数作为被除数和除数，返回他们相除结果的整数部分。

这个函数，功能就是 先`/` 再`floor`


    1. <?php
    2. //php 7
    3. echo intdiv(9,2), PHP_EOL;
    4. //之前的做法
    5. echo floor(9/2);



输出：


    1. 4
    2. 4



### 16. Session options (Session新增一个选项)

现在，`session_start()`函数可以接收一个数组作为参数，可以覆盖php.ini中session的配置项。比如，把`cache_limiter`设置为私有的、同时在阅读完session后立即关闭。

之前是在php.ini中设置，现在可以通过`session_start()`就可以了：


    1. <?php
    2. //加入参数
    3. session_start([
    4. 'cache_limiter'=>'private',
    5. 'read_and_close'=>true,
    6. ]);
    7. ?>



### 17. CSPRNG

新增了两个函数: `random_bytes()` and `random_int()`.可以产生被更加安全的随机整数和字符串。


    1. <?php
    2. //php 7 新增生成指定长度的随机字符串
    3. echo random_bytes(5), PHP_EOL;
    4. // php 7 新增生成指定长度的随机整数
    5. echo random_int(5,100), PHP_EOL;
    6. // 老版本php中的生成随机数
    7. echo mt_rand(5,100);



打印结果：


    1. ��,h,
    2. 32
    3. 52%




    1. -�wI.
    2. 5
    3. 9%



发现打印的随机字符串都是乱码啊。为了能够识别，可以转换成十六进制的。


    1. //php 7 新增生成指定长度的随机字符串，并转换成十六进制
    2. echo bin2hex(random_bytes(5)), PHP_EOL;



打印结果就好认了：


    1. 294bb036c1
    2. 94f3e0d5ad



### 18. preg_replace_callback_array()

新增了一个函数`preg_replace_callback_array()`，使用该函数可以使得在使用`preg_replace_callback()`函数时代码变得更加优雅。在PHP7之前，回调函数会调用每一个正则表达式，回调函数在部分分支上是被污染了。

OK了，搞懂了。麻痹昨天写了很多，居然同步失败了，又写了一遍！蛋。

[http://php.net/manual/zh/migration70.new-features.php](http://php.net/manual/zh/migration70.new-features.php)
[http://www.lanecn.com/article/main/aid-97](http://www.lanecn.com/article/main/aid-97)
[https://secure.php.net/manual/zh/migration70.incompatible.php](https://secure.php.net/manual/zh/migration70.incompatible.php)


[原文](https://www.zybuluo.com/phper/note/318273)
