# php基础

## require|require_once|include|include_once|use
* include 遇到文件错误产生一个警告但程序继续运行
* include_once 与include一样，但在代码已经被导入情况下不会继续导入
* require 遇到文件错误直接停止程序运行
* require_once 与require一样，但在代码已经被包括情况下不会继续包括
* use 按命名空间和类名导入

## php-cli|php-fpm
php-cli用于在命令行的php脚本执行或交互，而php-fpm则是一个进程管理器，接收web服务器的请求并处理php脚本再返回给web服务器\

## php执行过程
1.将php解释为tokens关键字\
2.通过tokens形成抽象的语法树AST，AST中是不同的节点集，并有先后优先级\
3.将AST转换为可以被ZendVM执行的Opcodes\
4.Zend引擎接收Opcodes并执行\

* Opcache
php扩展，缓存Opcodes以便跳过1、2、3步骤\
* JIT
php8中增加的新的即时编译器\
跳过ZendVM的编译过程，直接将Opcodes中的代码编译为cpu可执行语言进行执行\

## php不能在运行前编译的问题
因为php是弱类型语言，变量的类型会变动，而使用机器语言判断类型是不可行的，而且会很慢\
JIT通过增加名为DynASM的库，将特定格式的CPU指令映射为各种类型的CPU汇编代码，解决类型的问题\

## 版本更新

### 5.2
* json支持\

### 5.3
* 匿名函数\
实际是Closure类的一个实例\
在php中闭包和匿名函数一样，都是Closure类的实例\
```
$func = function($str){echo $str;}
$func('hello world');
```
* __invoke()\
在一个对象作为函数调用时调用\
```
class A{
    __invoke($str){
        echo $str;
    }
}
$a = new A;
$a('hello world');
```
* __callStatic()\
在调用一个不存在的静态方法时被调用\
```
class A{
    //__call 在一个对象上调用一个不存在的方法时调用
    __call($method, $arguments){
        echo "调用的{$method}方法不存在";
        print_r($arguments);
    }
    __callStatic($method, $arguments){
        echo "调用的{$method}方法不存在";
        print_r($arguments);
    }
}
```
* 命名空间\

### 5.4
* 数组简写\
* traits\

### 5.5
* yield\
```
function generator(){
    yield 1;
    yield 2;
}
foreach(generator() as $val){
    echo $val;
}
```
* list\
用于在一次操作中给一组变量赋值\
```
$array = [1,2];
list($a,$b) = $array;
```

### 5.6
* 可变参数函数\
```
function sum(...$args){
    $sum = 0;
    $sum = array_reduce($args,function($carry,$item){
        return $carry+$item;
    })
    return $sum;
}
```

### 7
* 新增标量类的参数和返回值的类型声明，如int、string、float、bool\
```
function test(int a):string{
    return "{$a}";
}
```
* null合并运算符|??\
```
$a??0;  //isset($a)?$a:0;
$a?:0;  //empty($a)?$a:0;
```
* 太空船操作符|<=>\
比较左边和右边的表达式，在小于、等于、大于时分别返回-1、0、1
```
$a <=> $b
```
* define定义常量数组
```
defint('ANIMALS',['dog','cat']);
echo ANIMALS[1];
```
* 匿名类\
```
interface Log{
    public function log(string $msg);
}
class Appli{
    private $log;

    public function getLog():Log{
        return $this->log;
    }

    public function setLog(Log $log){
        $this->log = $log;
    }
}

$a = new Appli;
$a->setLog(new class implements Log{
    public function log(string $msg){
        return $msg;
    }
})
```
* Unicode codepoint 转义语法\
```
echo "\u{9876}";    //输出汉字：顶
```
* Closure::call()\
将一个匿名函数绑定到一个对象上闭包并执行\
```
class Test{
    public $name = "gluge";
}
$a = function(){
    return $this->name;
}
echo $a->call(new Test);    //gluge
```
* 为反序列化unserialize()提供过滤\
可以反序列化为不完整类\
```
$data = unserialize($foo, ["allow_class"=>false]);  //反序列化为不完整类
$data = unserialize($foo, ["allow_class"=>["ClassName1","ClassName2]]); //除1和2外都反序列化为不完整类
$data = unserialize($foo, ["allow_class"=>true]);  //默认的反序列化，已经加载的类会反序列化为完整类，没有加载的类会反序列化为不完整类

$data_field = get_object_vars($data);   //可以通过这个方法获取不完整类的字段
```
* 新增IntlChar类\
用于操作unicode字符\
* use导入同命名空间下的多个类、方法、常量\
```
use namespace\{class1,class2}
use function namespace\{method1,method2}
use const namespace\{const1,const2}
```
* intp()\
相除取整，返回int\
```
intp(10,3)  //3,int
float(10,3)   //3,float
```
* random_bytes(int bytes)\
按字节数生成随机字符串，实际字符串长度为bytes*2\
```
random_bytes(4);    //e62a94a2
```
* random_int(min,max)\
在区间内生成随机整数\
```
random_int(10,99)   //11
```
* preg_replace_callback_array()\
使用一个数组进行正则表达式匹配\
```
$subject = 'Aaaaaa Bbb';
 
preg_replace_callback_array(
    [
        '~[a]+~i' => function ($match) {
            echo strlen($match[0]), ' matches for "a" found', PHP_EOL;
        },
        '~[b]+~i' => function ($match) {
            echo strlen($match[0]), ' matches for "b" found', PHP_EOL;
        }
    ],
    $subject
);

//6 matches for "a" found
//3 matches for "b" found
```
* session_start([options])\
session_start 可以接受参数作为配置，覆盖php.ini内的配置\
* 生成器有返回值，且可以在一个生成器中引入其他生成器\
```
function generator(){
    yield 1;
    yield 2;
    yield from generator2;
    return 3;
}
function generator2(){
    yield 1;
    yield 2;
}
$generator_class = generator();

foreach(generator_class as $val){
    echo $val;
}
echo $generator_class->getReturn();

//12123
```
* foreach 不再移动数组指针，且遍历时有更好的迭代性\
```
$arr = [0];
foreach($arr as &$v){
    $arr[1] = 1;
    echo $v;
}
//01
```
* 十六进制字符串不再认为是数字\
* 数值溢出时返回null\
* ini文件注释使用;而非#\

### 8
* 联合类型
方法的参数类型和返回类型可以定义联合类型\
```
function test(int|float num):int|float{
    return num;
}
```
* 新增WeakMap
可以创建对象到值的映射的集合，在对象存在时集合内存在映射，在对象消失时对应映射会消失\
可以理解为键为弱引用的数组\
```
class Foo{
    public WeakMap $weak;
    public __construc(){
        $this->cache = new WeakMap();
    }
    public function getCache(object $obj){
        return $this->cache[$obj] ??= $this->generatorValue();
    }
    public function generatorValue(){
        return random_int(1,100);
    }
}
$obj = new stdClass();
$cache = new Foo;
$cache->getCache($obj);
var_dump(count($cache->cache));     //1
unset($obj);
var_dump(count($cache->cache));     //0
``` 
* 新增ValueError异常类
当传入方法的参数不符合方法的设定时会抛出\

可以用可变参数重写方法\
```
class A{
    public function method(int $many, string $params){

    }
}
class B{
    public function method(...$every){
        var_dump($every);
    }
}
$b = new B();
$b->method('zzzz');     //array 0=>zzzz
```

method():static\
返回方法所属的类\
```
class A{
    public function method():static{
        return $this;
    }
}
```
* obj::class 返回类型
等同于get_class($obj)\
```
class Test{
    
}

$t = new Test;
var_dump($t::class);
```
* new、instance 可以用于表达式
```
class Test{}

$names = ['Test'];
$t = new ($names[0]);
```
* StringAble接口
实现了__toString()方法的类就会被认为实现了该接口\
```
class Test{
    public function __toString(){
        return 'i am test';
    }
}
```
* trait 可以定义私有抽象方法
使用的类需要实现方法\
```
trait myT{
    abstract private function needed():string;
}

class Test{
    use myT;

    private function needed():string{
        return '';
    }
}
```
* throw 可用于表达式，如??、?:等
```
$a ?? throw new \Exception();
```
* 只捕获异常不存储到变量
```
try{

}catch (\Exception){
    echo "doSomethin";
}
```
* 新增mixed类型定义，等价于array|object|bool|int|float|string|null|callable|resource
```
public function method(mixed ...$data){

}
```
* 注解，就是c#中的特性，可以为类、函数添加元数据
元数据可以通过反射获取\
```
#[Attribute]
class myAttribute{
    public array $params = [];
    public function __construct(...$args){
        $this->params = $args;
    }
}

#[myAttribute('ddd')]
class Test{

}

//通过反射获取添加的元数据
$reflect = new ReflectionClass(Test::class);
$attributes = $reflect->getAttributes(myAttribute::class);
foreach($attributes as $attribute){
    $attribute = $attribute->newInstance();
    var_dump($attribute->params);
}
```
* 在需要字段和构造函数时可以简化书写
```
class User{
    public function __construct(public int $age, public string $name){

    }
}

$a = new User(1,'haha');
echo $a->age;
echo $a->name;
```
* match
类似于switch\
```
$a = match(1){
    0=>'aa',
    1=>'bb',
}
echo $a;    //bb
```
* ?->
对空安全运算符，在左不为null时按正常->运行，左侧为null直接返回null\
```
class A{
    public function getAddress(){

    }
}

$a = new A;
$phone = $a->getAddress->phone;
```
* 命名参数
可以以参数名:值的形式传递参数，并且可以跳过默认参数\
```
array_fill(start_index:0, num:100, value: 50);
```

* Jit
优化Opcodes被ZendVm编译为机器语言的过程\


## 数组函数
* array_reduce(array,func,mixed)
用回调函数将数组归一化为单一的值\
mixed表示在第一个数组处理前，数组的初始值\
```
$arr = ['aaa','bbb','ccc'];
$res = array_reduce($arr, function($carry,$item){
    return $carry.$item;
},'mid_');
//$res mid_aaabbbccc

$res = array_reduce($arr, function($carry,$item){
    return function() use($carry,$item){
        var_dump($item);
        if(is_null($carry)){
            var_dump("no carry");
        }elseif($carry instanceof \Closure){
            var_dump($carry());
        }
        return strtoupper($item);
    }
})
var_dump($res());
//ccc
//bbb
//aaa
//nocarray
//AAA
//BBB
//CCC
```
* array_reverse(array)
返回数组的倒序数组\



## 字符串函数
* strlen($str)
获取字符长度



# Laravel

## 生命周期
* composer加载依赖项
* 创建应用实例
创建容器\
绑定内核，包括http内核（处理正常请求）、Console内核（处理artisan、计划任务、队列等）、异常处理\
* 接受请求并响应
解析http内核\
处理http请求，将请求参数创建一个request实例传入http内核的handle方法，具体是将request实例注册到app容器，清除原request实例缓存，启动引导程序（检查环境、加载配置、配置日志、注册异常处理handler、注册facades、注册并启动providers），将请求发送到路由\
发送响应\
* 终止应用程序
调用中间件处理session等内容\
调用terminate方法\

## 服务容器
执行依赖注入的工具\

## 事件
在laravel中触发活动时会自动触发\

## 中间件的实现原理
使用array_reduce返回中间件执行的嵌套闭包\
由于array_reduce返回的闭包的最外层是传入array的最后一个元素，而最外层闭包又是最先执行的，因此先通过array_reverse将中间件数组倒序，以获得正序执行的嵌套闭包\



# 依赖注入

## 构造函数注入
从构造器注入\
```
class test{

    private $class;

    public function __construct(class $class){
        $this->class = $class;
    }
}
```

## setter注入
编写特定的set方法注入\
```
class test{

    private $class;

    public function setClass(class $class){
        $this->class = $class;
    }
}
```

## 接口注入
类实现接口，接口中某方法参数就是需要注入的内容\
由于实现接口功能反而需要类中扩展很多代码，因此不建议使用\



# PHP RPC的实现

## 原生实现
```
$rpc_server = stream_socket_server("tcp://127.0.0.1:6661",$err_no,$err_msg);
if(!$rpc_server){
    echo "代码异常：".$err_no.",信息异常：".$err_msg;
    exit();
}
while(true){
    $result = [];//定义返回结果数组
    try{
        $buff = stream_socket_accept($rpc_server);
        $data = fread($buff,2048);//读取客户端选数据
        $json = json_decode($data,true);//转换客户端的json数据
        $class = $json['class'];//客户端需要访问的类
        $file = $class.".php";
        require_once $file;//包含进来
        if(!file_exists($file)){//如果文件不存在
            throw new Exception('文件不存在','404');
        }
        $method = $json['method'];//客户端需要访问的方法
        $obj = new $class();//实例化对象
        $rpc_server_data = $obj->$method($json['can_shu']);//调用方法
        $result['code'] = 1;//成功返回1
        $result['data'] = $rpc_server_data;//返回数据
        $result['msg'] = "请求成功";
        $result_data = json_encode($result);//转换为json数据
        fwrite($buff,$result_data);
        fclose($rpc_server);//关闭资源  
    }catch(Excepiton $e){
        $result['code'] = $e->getCode();//失败返回
        $result['data'] = $rpc_server_data;//返回空数据
        $result['msg'] = $e->getMessage();//失败返回
        $result_data = json_encode($result);//转换为json数据
        fwrite($buff,$result);
        fclose($rpc_server);//关闭资源
    }
}
```

## Hprose
可以实现RPC传输的包\
```
<?php
require 'vendor/autoload.php';

use Hprose\RPC\Http\HttpServer;
use Hprose\RPC\Service;

function hello(string $name): string {
    return "Hello " . $name . "!";
}

$service = new Service();
$service->addCallable("hello", "hello");
$server = new HttpServer();
$service->bind($server);
$server->listen();
```
```
<?php
require 'vendor/autoload.php';

use Hprose\RPC\Client;
use Hprose\RPC\Plugins\Log;

$client = new Client(['http://127.0.0.1:8024/']);
$log = new Log();
$client->use([$log, 'invokeHandler'], [$log, 'ioHandler']);
$proxy = $client->useService();
$result = $proxy->hello('world');
print($result);
```