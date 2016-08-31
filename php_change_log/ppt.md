title: PHP5.2自5.6新特性
speaker: Dray
url: https://www.jaylee.cc/php5-2-since-new-in-5-6/
transition: cards
files: /js/demo.js,/css/demo.css

[slide]

截至目前，PHP的最新稳定版本是PHP5.6，我们现在用的是 php5.4。

[slide]

**本文会介绍自PHP5.2起，直到PHP5.6中增加的新特性。**

*   PHP5.2以前：autoload, PDO和MySQLi, 类型约束
*   PHP5.2：JSON的支持
*   PHP5.3：弃用的功能，匿名函数，新增魔术方法，命名空间，延迟静态绑定，Heredoc和Nowdoc，const，三元运算符，Phar
*   PHP5.4：Short Open Tag，数组简写形式，Traits，内置web服务器，细节修改
*   PHP5.5：yield，list()可用于foreach循环，细节修改
*   PHP5.6：常量增强，可变函数参数，命名空间增强

[slide]

> PHP5.2以前
> 
> 2006年前（顺便介绍一下PHP5.2已经出现但是值得介绍的特性）

## autoload

大家应该都知道__autoload()函数，如果定义了该函数，那么当在代码中使用了一个未定义的类的时候，该函数就会被调用，你可以在该函数中加载相应的类实现文件，如：



```
<?php

    function __autoload($className){
        require_once $className.".class.php";
    }

    $foo = new Foo();

?>

```



这样，在执行new操作的时候，会在当前目录下寻找Foo.class.php这个文件，并加载进来。

[slide]

但该函数已经不建议使用，原因是一个项目中仅能有一个这样的__autoload()函数，因为PHP不允许函数重名。但当你用到一些类库的时候，难免会出现多个autoload函数的需要，于是spl_autoload_register()取而代之：



```
<?php    

    function autoloadModel($className){
        $filename = "models/".$className.".php";
        if ( file_exists($filename) ){
            require_once $filename;
        }
    }

    function autoloadController($className){
        $filename = "controller/".$className.".php";
        if ( file_exists($filename) ){
            require_once $filename;
        }
    }

    spl_autoload_register("autoloadModel");
    spl_autoload_register("autoloadController");

?>

```



spl_autoload_register()会将一个函数注册到autoload函数列表中，当出现未定义的类的时候，SPL会按照注册的顺序逐个调用被注册的autoload函数，这意味着你可以使用spl_autoload_register注册多个autoload函数。

[slide]

## PDO和MySQLi

PDO即PHP Data Object , PHP数据对象，这是PHP的新式数据库访问接口。

按照传统的风格，访问MySQL数据库应该是这个样子：

```
<?php

    $conn = mysql_connect("localhost","user","passwd") or die("Connection failed");

    mysql_select_db("test");

    $type = $_POST["type"];

    $sql = "select * from `table` where `type` = {$type}";

    $result = mysql_query($sql, $conn);

    while($row = mysql_fetch_assoc($result)){
        var_dump($row);
    }

    mysql_free_result($result);

    mysql_close($conn);

?>

```

[slide]

为了能让代码实现与数据库无关，即同一段代码适用于多种数据库（例如以上代码仅适用于MySQL），PHP官方设计了PDO.

除此之外，PDO还提供了更多的功能，比如：

*   面对对象风格接口
*   SQL预编译，占位符语法
*   更高的执行效率，作为官方推荐，有特别的性能优化
*   支持大部分SQL数据库，更换数据库无需改动代码

[slide]

上面的代码用PDO实现将会是这个样子：

```
<?php

    try {

        $dsn = "mysql:host=localhost;dbname=test";

        $conn = new PDO($dsn, "user", "passwd");

        $sql = "select * from user where type = :type ";

        $stmt = $conn->prepare($sql);

        $stmt->bindParams("type",$_POST["type"]);

        $stmt->execute();

        $stmt->fetchAll(PDO::FETCH_ASSOC);

    }catch(PDOException $e){

        echo $e->getMessage();
    }

?>

```



[slide]

PDO是官方推荐的，更为通用的数据库访问方式，如果你没有特殊的需求，那么最好学习和使用POD，但如果你需要使用MySQL所特有的高级功能，那么你可能需要尝试一下MySQLi，因为PDO为了能够同时在多种数据库上使用，不会包含那些MySQL独有的功能。

[slide]

## 类型约束

通过类型约束可以限制参数的类型，不过这一机制并不完善，仅适用于对象，接口，callable，以及array，不适用于string和int，如果参数类型不匹配，则会产生一个fatal error.


```
<?php

    function test(callable $callable, array $arr, MyClass $myclass){
        //...
    }

?>

```



[slide]

> PHP5.2
> 
> 2006年 ~ 2011年

## JSON支持

包括json_encode(), json_decode()等函数，JSON算是在web领域非常实用的数据交换格式，可以被JS直接支持，JSON实际上JS语法的一部分。

JSON系列函数，可以将PHP中的数组结构与JSON字符串进行转换。

[slide]

```
<?php

    $arr = array(
        "key"   =>  "value",
        "array" =>  array(1,2,3,4)
    );

    $json = json_encode($arr);

    echo $json;

    $object = json_decode($json);   

    print_r($object);
?>

```

[slide]

输出：

```
    {"key":"value","array":[1,2,3,4]}

    stdClass Object
    (
        [key] => value
        [array] => Array
            (
                [0] => 1
                [1] => 2
                [2] => 3
                [3] => 4
            )
    )

```

[slide]

值得注意的是，json_decode()默认会返回一个对象而非数组，如果需要返回数组需要将第二个参数设为true。同时json_encode()在php5.4的时候增加了一个**JSON_UNESCAPED_UNICODE**的常量，这样，如果键值对中包含中文就不会被编码成unicode字符了。

[slide]

> PHP5.3
> 
> 2009年 ~ 2012年 PHP5.3算是一个非常大的更新 ，新增了大量的新特性，同时也做了一些不向下兼容的修改。

## 弃用的功能

以下几个功能被弃用，若在配置文件中启用，则PHP在运行时会发出警告。

*   `Register Globals`

这是php.ini中的一个选项(register_globals)，开启后会将所有表单变量($_GET 和 $_POST）注册为全局变量。

[slide]

看下面的例子：

```
<?php

    if ( isAuth() ){
        $autherize = true;
    }

    if ($autherize){
        include("page.php");
    }

?>

```

[slide]

这段代码在通过验证时，将$autherize设为true，然后根据$autherize的值来决定是否显示页面。

但由于没有事先把$authorize初始化为false，当register_globals打开时，可能访问/auth.php?authorize=1来定义该变量值，绕过身份验证。

该特征属于历史遗留原因，在PHP4.2中默认被关闭，在PHP5.4中被移除。

[slide]


*   `Magic Quotes`

对应php.ini中的选项 magic_quotes_gpc，这个特征同样属于历史遗留原因，已经在PHP5.4移除。

该特征会对所有用户输入的 ‘ (单引号)，“ (双引号)，\(反斜线) 进行转义，和addslashes()作用完全一样，这看上去不错，但是PHP并不知道哪些输入会进SQL，哪些输入会进Shell，哪些输入会被显示为HTML，所以很多时候这种转义会引起混乱。

[slide]

PHP一共有三个魔术引号指令：

magic_quotes_gpc：影响HTTP请求数据(GET, POST和COOKIE)，不能在运行时改变，在PHP中默认值为On。( 相关函数：get_magic_quotes_gpc() )

magic_quotes_runtime：如果打开的话，大部分从外部来源取得数据并返回的函数，包括从数据库和文件，所返回的数据都会被转义。该选项可在运行时改变，在PHP中默认值为Off。( 相关函数：get_magic_quotes_runtime()，set_magic_quotes_runtime() )

magic_quotes_sybase：如果打开的话，将会使用单引号对单引号进行转义而不是反斜线。此选项会完全覆盖magic_quote_gpc。( 获取该选项的值通过ini_get()函数 )

[slide]

*   `Safe Model`

很多虚拟主机提供商使用Safe Mode 来隔离多个用户，但Safe Model存在诸多问题，例如有些扩展并不按照Safe Mode来进行控制。

PHP官方推荐使用操作系统的机制来进行权限隔离，让Web服务器以不同的用户权限来运行PHP解释器。

[slide]

## 匿名函数

匿名函数也叫闭包(Closures)，经常被用来临时性的创建一个无名函数，用于回调函数等用途。



```
<?php

    $func = function(){
        var_dump(func_get_args());
    }

    $func("Hello","world");
?>

```

以上代码定义了一个匿名函数，并赋值给了func。

可以看到定义匿名函数依然使用function关键字，只不过省略了函数名，直接是参数列表。

然后我们又调用了$func所储存的匿名函数。

[slide]

匿名函数还可以通过use关键字来捕捉外部变量。

```
<?php

    function ArrayPlus($array, $num){

        array_walk($array, function(&$v) use($num){
            $v *= $num;
        });

        return $array;
    }

    var_dump(ArrayPlus([1,2,3,4,5], 6));
?>

```


[slide]

上面的代码定义了一个ArrayPlus()函数（这不是匿名函数），它会将一个数组（$array）中的每一项，加上一个指定的数字（$num）。

在ArrayPlus()的实现中，我们使用了array_walk()函数，它会为一个数组的每一项执行一个回调函数，即我们定义的匿名函数。

在匿名函数的参数列表后，我们用use关键字将匿名函数外的$num捕捉到了函数内部，以便我们知道该加多少。

[slide]

## 魔术方法: __invoke(), __callStatic()

PHP的面向对象体系中，提供了若干“魔术方法”，用于实现类似其它语言中的“重载”，如访问不存在的方法、属性时触发某个魔术方法。

随着匿名函数的加入，PHP引入了一个新的魔术方法__invoke()。
當嘗試以調用函數的方式調用一個對像時，__invoke 方法會被自動調用

```
<?php

    class A {

        public function __invoke($str){
            echo "A::__invoke:{$str}";
        }
    }

    $a = new A();

    $a("jaylee.cc");    //A::__invoke:jaylee.cc
?>

```

[slide]

__callStatic()则会在调用一个不存在的静态方法时被调用。

```
    <?php

    class A {

        public function __callStatic($methodName, $args){
            var_dump($methodName, $args);
        }
    }

    A::test("aa","bb");

?>

```

[slide]

## 命名空间

```
<?php

    // 命名空间的分隔符是反斜杠，该声明语句必须在文件第一行。
    // 命名空间中可以包含任意代码，但只有类，函数，常量受命名空间的影响

    namespace Jaylee\Test;

    // 该类的完整限定名是\Jaylee\Test\A，其中第一个反斜杠表示全局命名空间
    class A {
        public function test(){
            echo "namespace:".__NAMESPACE__.", ".__CLASS__."<br />";
        }
    }

    // 你还可以在命名空间中定义第二个命名空间，接下来的代码都位于\Other\Test2
    namespace Other\Test2;

    // 实例化来自其它命名空间中的对象
    $a = new \Jaylee\Test\A;

    class B {
        public function test(){
            echo "namespace:".__NAMESPACE__.", ".__CLASS__."<br />";        
        }
    }

    namespace Other;

    // 实例化来自子命名空间的对象        
    $b = new Test2\B;

    $b->test();

    // 导入来自其它命名空间的名称，并重命名
    // 注意只能导入类，不能用于函数和常量
    use \Jaylee\Test\A as ClassA;

    $a = new ClassA();

    $a->test();    
?>

```

[slide]

更多有关命名空间的语法介绍请参见[官网](http://php.net/manual/zh/language.namespaces.rationale.php)。

命名空间经常和autoload一起使用，用于自动加载类文件：


```
<?php

    spl_autoload_register(
        function($className){
            spl_autoload(str_replace("\\", "/", $className));
        }
    );

?>

```

当你实例化一个类\Jaylee\Test\TestA的时候，这个类的完整限定名称会被传递给autoload函数，autoload函数将类名中的命名空间分隔符替换为反斜杠，并包含对应文件。

这样可以实现类定义文件分组存储，按需自动加载。

[slide]

## 延迟静态绑定

PHP的的 OPP 机制，具有继承和类似虚函数的功能，例如如下的代码：

```
<?php

    class A {

        public function callFoo(){
            echo $this->foo();
        }

        public function foo(){
            return "A::foo()";
        }
    }

    class B extends A {

        public function foo(){
            return "B::foo()";
        }
    }

    $b = new B();

    $b->callFoo();      //B::foo();
?>

```

[slide]

可以看到，当在A中使用了`$this->foo()`时，体现了“虚函数”的机制，实际调用的是B::foo()，然后如果将所有的函数都改为静态函数时：

```
<?php

    class A {

        static public function callFoo(){
            echo self::foo();
        }

        static public function foo(){
            return "A::foo()";
        }
    }

    class B extends A {

        static public function foo(){
            return "B::foo()";
        }
    }

    B::callFoo();   //A::foo()
?>

```

[slide]

这时，输出的会是`A::foo()`，这是因为self的语义本来就是“当前类”，所有在PHP5.3给static关键赋予了一个新的功能：延迟静态绑定：

```
<?php

    class A {

        static public function callFoo(){
            echo static::foo();
        }

        //...
    }

?>

```

将self改为static之后，就会像预期一样输出`B::foo`了。

[slide]

## Heredoc 和 Nowdoc

PHP5.3对Heredoc以及Nowdoc进行了一些改进，它们都用于在PHP中嵌入大段的代码。

Heredoc的行为类似于一个双引号字符串：

```
<?php

$name = "Jaylee";

echo <<<TEXT

my name is "{$name}"

TEXT;

?>

```


Heredoc以三个尖括号开始，后面跟一个标识符（TEXT）,直到一个同样的定格标识符（不能缩进）结束。 就像双引号字符串一样，其中可以嵌入变量。

[slide]

Heredoc还可以用于函数参数，以及类成员初始化：

```
<?php

var_dump(<<<EOD
    hello world
EOD
);

class A {

    const name = <<<EOD
hello world
EOD;

    public $foo = <<<EOD
hello world2
EOD;

}

?>

```

[slide]

Nowdoc的行为像一个单引号字符串，不能在其中嵌入变量，和Heredoc唯一的区别就是，三个尖括号的标识符要用单引号引起来：

```
<?php

    $name = "Jaylee";

    echo <<<'EOD'

my name is "{$name}"

EOD;

    //输出：my name is "{$name}"

?>

```

[slide]

## 用const定义常量

php5.3起同时支持在全局命名空间和类中使用 const 定义常量。

旧式风格：

```
<?php

    define("NAME","jaylee");

?>

```

[slide]

新式风格：

```
<?php

    const NAME = "Jaylee";

?>

```

[slide]

const形式仅适用于常量，不适用于运行时才能求值的表达式。

```
<?php

    const VALUE = 1234; //正确

    const VALUE = 1000*2; //错误
?>

```

[slide]

## 三元运算符的简写形式

旧式风格：

```
<?php

    echo $a ? $a : "No VALUE";

?>

```

可以简写成：

```
<?php

    echo $a ? : "No Value";

?>

```

即如果省略三元运算符的第二个部分，会默认用第一个部分代替。

[slide]

## Phar

Phar即PHP Archive, 起初只是Pear中的一个库而已，后来在PHP5.3被重新编写成C扩展并内置到 PHP 中。 Phar用来将多个 .php 脚本打包(也可以打包其他文件)成一个 .phar 的压缩文件(通常是ZIP格式)。 目的在于模仿 Java 的 .jar, 不对，目的是为了让发布PHP应用程序更加方便。同时Phar还提供了数字签名验证等功能。

.phar 文件可以像 .php 文件一样被 PHP 引擎解释执行，同时你还可以写出这样的代码来包含 .phar 中的代码。

```
<?php

    require "xxx.phar";
    require "phar://xxx.phar/aa/bb.php";

?>

```

更多信息请参见[官网](http://www.php.net/manual/zh/phar.using.intro.php);

[slide]

> PHP5.4
> 
> 2012 ~ 2013

## Short Open Tag

Short Open Tag 自PHP5.4起总是可用。

在这里集中讲一下有关 PHP 起止标签的问题。即：

```
<?php

    //code

?>

```

[slide]

通常就是上面的形式，除此之外还有一种简写形式：

```
<? /*  code  */ ?>

```

还可以把

```
<? echo $foo;?>

```

简写成：

```
<?=$foo?>

```

[slide]

这种简写形式被称为 Short Open Tag, 在 PHP5.3 起被默认开启，在 PHP5.4 起总是可用。 使用这种简写形式在 HTML 中嵌入 PHP 变量将会非常方便。

对于纯 PHP 文件(如类实现文件), PHP 官方建议顶格写起始标记，同时 `省略` 结束标记。 这样可以确保整个 PHP 文件都是 PHP 代码，没有任何输出，否则当你包含该文件后，设置 Header 和 Cookie 时会遇到一些麻烦（Header 和 Cookie 必须在输出任何内容之前被发送）。

[slide]

## 数组简写形式

这是非常方便的一项特征！

```
<?php

    //原来数组的写法
    $arr = array("key" => "value", "key2" => "value2");

    //简写形式
    $arr = ["key" => "value", "key2" => "value2"];

?>

```

[slide]

## Traits

所谓Traits就是“构件”，是用来替代继承的一种机制。PHP中无法进行多重继承，但一个类可以包含多个Traits.

```
// Traits不能被單獨實例化，只能被類所包含
trait SayWorld
{
    public function sayHello()
    {
        echo 'World!';
    }
}

class MyHelloWorld
{
    // 將SayWorld中的成員包含進來
    use SayWorld;
}

$xxoo = new MyHelloWorld();
// sayHello() 函數是來自 SayWorld 構件的
$xxoo->sayHello();
```

[slide]

Traits还有很多神奇的功能，比如包含多个Traits, 解决冲突，修改访问权限，为函数设置别名等等。
Traits中也同样可以包含Traits. 篇幅有限不能逐个举例，详情参见官网 [注].

注：http://www.php.net/manual/zh/language.oop5.traits.php

[slide]

## 内置 Web 服务器

PHP从5.4开始内置一个轻量级的Web服务器，不支持并发，定位是用于开发和调试环境。

在开发环境使用它的确非常方便。

```
php -S localhost:8000

```

这样就在当前目录建立起了一个Web服务器，你可以通过 http://localhost:8000/ 来访问。

[slide]

其中localhost是监听的ip，8000是监听的端口，可以自行修改。

很多应用中，都会进行URL重写，所以PHP提供了一个设置路由脚本的功能:

```
php -S localhost:8000 index.php

```


这样一来，所有的请求都会由index.php来处理。

[slide]

## 细节修改

**PHP5.4 新增了动态访问静态方法的方式：**

```
<?php

    $func = "Foo";

    A::{$func}();   // 相当于 A::Foo();

?>

```

[slide]

**新增在实例化时访问类成员的特征：**

```
<?php

    (new MyClass())->foo();

?>

```

[slide]

**新增支持对函数返回数组的成员访问解析(这种写法在之前版本是会报错的)：**

```
<?php

    var_dump( func()[0] );      //如果func返回一个数据，这里直接取第0项元素

?>

```

[slide]

> PHP5.5
> 
> 2013起

## yield关键字

yield关键字用于当函数需要返回一个迭代器的时候，逐个返回值。

该函数返回一个迭代器对象。

```
function number10()
{
    for($i = 1; $i <= 10; $i += 1)
        yield $i;
}
```

该函数的返回值是一个数组：

```
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

[slide]

## list()用于foreach

该特性可以在foreach中解析嵌套的数组：

```
<?php

    $arr = [
        [1,2,3],
        ["a","b","c"]
    ];

    foreach ($arr as list($a, $b, $c)){
        echo $a."==".$b."==".$c.PHP_EOL;
    }

?>

```

结果：

```
    1==2==3
    a==b==c

```

[slide]

## 密码哈希

新的密码哈希API是PHP5.5中非常重要和实用的特性。以前，开发者只能依赖其他的crypt()函数，而这些函数的文档又不是很齐全，导致很多误用。 现在新增的API简单明了，方便开发者实现安全的密码哈希功能。

新API包含password_hash()和password_verify()等函数，调用password_hash($password, PASSWORD_DEFAULT)返回一个使用bcrypt加密，并自动添加 了salting的哈希值。验证密码的时候则调用password_verify($password, $hash)。API现在默认使用bcrypt，将来可能会引入其他新的更安全的加密 方式。开发者可以自己调整bcrypt的参数来提高加密强度，可以自己指定salt值等（但是官方不建议这么做）。

[slide]

## finally

PHP5.5开始支持其他语言的异常处理中常用的finally关键字，开发者可以从此在try和catch块之后，运行指定代码，而无需关心是否有异常抛出，然 后再回到正常执行流，而在此之前，开发者只能在try和catch块中拷贝代码，来完成相关的任务清理工作。比如下面的例子，必须在两个地方 调用releaseResource()：

```
<?php
function doSomething() {
    $resource = createResource();
    try {
        $result = useResource($resource);
    }
    catch (Exception $e) {
        releaseResource($resource);
        log($e->getMessage());
        exit();
    }
    releaseResource($resource);
    return $result;
}
```

有了finally关键字后，就可以删除冗余代码：

```
<?php
function doSomething() {
    $resource = createResource();
    try {
        $result = useResource($resource);
        return $result;
    }
    catch (Exception $e) {
        log($e->getMessage());
        exit();
    }
    finally {
        releaseResource($resource);
    }
}
```

修改后的代码，我们只需在finally块中调用清理函数releaseResource()，无论流程最终是走到try中的return语句，还是到catch中的exit，finally 中的代码都会执行。

[slide]

## 数组和字符串字面量解引用

现在访问数组的语法中，支持数组和字符串字面量的解引用：
```
<?php
// array dereferencing - returns 3
echo [1, 3, 5, 7][1];

// string dereferencing - returns "l"
echo "hello"[3];
```

这个特性主要是增强了语言的一致性，对我们平时写代码的行为可能影响不大，但是在某些情景下使用还是非常便利的：

```
<?php
$randomChar = "abcdefg0123456789"[mt_rand(0, 16)];
```

[slide]

## empty()支持函数调用和表达式

empty()这个语言结构开始支持在函数调用和表达式中使用，如empty($object->getProperty())。这样就可以用empty()判断函数返回值，而不 用先把返回值赋值给一个临时变量，然后对临时变量使用empty()。

[slide]

## 类名解析
从PHP5.3引入命名空间之后，使用它来组织PHP项目中的类结构已经司空见惯，但是要取回带命名空间的全限定类名，却是非常的困难，如：

```
<?php
use Namespaced\Class\Foo;

$reflection = new ReflectionClass("Foo");
```

这个代码将会失败，因为它会从全局命名空间中查找类Foo，而不是指定的命名空间。PHP5.5引入了class关键字，可以通过它获取全限定类名：

```
<?php
use Namespaced\Class\Foo;

$reflection = new ReflectionClass(Foo::class);
```
上面的代码中，Foo:class会解析为"Namespaced\Class\Foo"。

[slide]

## 其它细节修改

*   不推荐使用 mysql 函数，推荐使用 PDO 或 MySQLi, 参见前文。

*   不再支持Windows XP.

*   可用 MyClass::class 取到一个类的完整限定名(包括命名空间)。

*   empty() 支持表达式作为参数。

[slide]

> PHP5.6
> 
> 2014年8月

## 常量表达式

在常量、属性声明和函数参数默认值声明时，以前版本只允许常量值，PHP5.6开始允许使用包含数字、字符串字面值和常量的标量表达式。


```
<?php

    const ONE = 1;
    const TWO = ONE * 2;

    class C {

        const THREE = TWO + 1;

        const ONE_THIRD = ONE / self::THREE;

        const SENTENCE = 'The value of '.self::THREE.' is 3';

        public function f ($a = ONE + self::THREE) {
            return $a;
        }
    }

    echo (new C)->f();  // 4

    echo C::SENTENCE;   //The value of 3 is 3

?>

```

[slide]

## 可变参数函数

可变函数的实现，不再依赖`func_get_args()`函数，现在可以通过新增的操作符`...`更简洁地实现。

```
<?php

    function foo($req, $opt = null, ...$params){

        printf("\$req: %d; \$opt: %d; number of params: %d\n",
                $req, $opt, count($params));
    }

    foo(1);
    foo(1,2);
    foo(1, 2, 3);
    foo(1, 2, 3, 4);
    foo(1, 2, 3, 4, 5);

?>

```

[slide]

以上输出：

```
    $req: 1; $opt: 0; number of params: 0
    $req: 1; $opt: 2; number of params: 0
    $req: 1; $opt: 2; number of params: 1
    $req: 1; $opt: 2; number of params: 2
    $req: 1; $opt: 2; number of params: 3

```

[slide]

## 参数解包功能

在调用函数时，通过`...`操作符可以把数组或者可遍历对象解包到参数列表，这和Ruby等语言中的扩张(splat)操作符类似。


```
<?php

    function add($a, $b, $c){
        return $a + $b + $c;
    }

    $operators = [1, 2, 3];

    echo add(...$operators);    //6
?>

```

[slide]

## 导入函数和常量

use 操作符开始支持函数和常量的导入。 `use function` 和 `use const` 结构的用法的示例：

```
<?php

    namespace cc\jaylee {

        const FOO = 11;

        function f(){
            echo __FUNCTION__;
        }
    }

    namespace {

        use const cc\jaylee\FOO;
        use function cc\jaylee\f;

        echo FOO;   \\ 11

        f();        \\ cc\jaylee\f
    }

?>

```

[slide]

## phpdbg

PHP自带了一个交互式调试器phpdbg，它是一个SAPI模块，更多信息参考 [phpdbg 文档](http://phpdbg.com/docs)。

## php://input可以被复用

`php://input` 开始支持多次打开和读取，这给处理POST数据的模块的内存占用带来了极大的改善。

## 大文件上传支持

可以上传超过2G的大文件。

[slide]

## GMP支持操作符重载

GNU多重精度運算庫（英語：GNU Multiple Precision Arithmetic Library，簡稱GMP或gmpal）是一個開源的任意精度運算庫，支持正負數的整數、有理數、浮點數。它沒有任何任何精度限制，只受限於可用內存。GMP有很多函數，它們都有一個規則的接口。它是C語言寫成的，但用為其他很多語言做包裝，包括Ada，C++，C#，OCaml，Perl，PHP和python。

GMP对象支持操作符重载和转换为标量，改善了代码的可读性，如：
```
<?php
$a = gmp_init(42);
$b = gmp_init(17);
 
// Pre-5.6 code:
var_dump(gmp_add($a, $b));
var_dump(gmp_add($a, 17));
var_dump(gmp_add(42, $b));

// New code:
var_dump($a + $b);
var_dump($a + 17);
var_dump(42 + $b);
```

[slide]

## 新增gost-crypto哈希算法

采用CryptoPro S-box tables实现了gost-crypto哈希算法，详情参考[RFC 4357, section 11.2。](http://www.faqs.org/rfcs/rfc4357)

[slide]
## SSL/TLS改进

OpenSSL扩展新增证书指纹的提取和验证功能，openssl_x509_fingerprint()用于提取X.509证书的指纹，SSL stream context 选项: capture_peer_cert 用于获取对方X.509证书；peer_fingerprint用于断言对方证书和给定的指纹匹配。

另外，可以通过SSL流上下文选项crypto_method指定加密方法，如SSLv3或TLS，目前支持的选项值包括STREAM_CRYPTO_METHOD_SSLv2_CLIENT, STREAM_CRYPTO_METHOD_SSLv3_CLIENT, STREAM_CRYPTO_METHOD_SSLv23_CLIENT (默认), or STREAM_CRYPTO_METHOD_TLS_CLIENT。

[slide]

参考文章：
[PHP5.2自5.6新特性](https://www.jaylee.cc/php5-2-since-new-in-5-6/)
[PHP 自 5.2 到 5.6 中新增的功能详解](https://segmentfault.com/a/1190000000403307)
[PHP5.5新特性介绍](http://wulijun.github.io/2013/07/17/whats-new-in-php-5-5.html)
[PHP5.6新特性介绍](http://wulijun.github.io/2014/01/25/whats-new-in-php-5-6.html)
