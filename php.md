# php基础

## require|require_once|include|include_once|use
* include\
遇到文件错误产生一个警告但程序继续运行
* include_once\
与include一样，但在代码已经被导入情况下不会继续导入
* require\
遇到文件错误直接停止程序运行
* require_once\
与require一样，但在代码已经被包括情况下不会继续包括
* use\
按命名空间和类名导入

## php-cli | php-fpm
php-cli用于在命令行的php脚本执行或交互，而php-fpm则是一个进程管理器，接收web服务器的请求并处理php脚本再返回给web服务器

## php执行过程
1. 将php解释为tokens关键字
2. 通过tokens形成抽象的语法树AST，AST中是不同的节点集，并有先后优先级
3. 将AST转换为可以被ZendVM执行的Opcodes
4. Zend引擎接收Opcodes并执行

* Opcache\
php扩展，缓存Opcodes以便跳过1、2、3步骤
* JIT\
php8中增加的新的即时编译器\
跳过ZendVM的编译过程，直接将Opcodes中的代码编译为cpu可执行语言进行执行

## php不能在运行前编译的问题
因为php是弱类型语言，变量的类型会变动，而使用机器语言判断类型是不可行的，而且会很慢\
JIT通过增加名为DynASM的库，将特定格式的CPU指令映射为各种类型的CPU汇编代码，解决类型的问题

## 版本更新

### 5.2
* json支持

### 5.3
* 匿名函数\
实际是Closure类的一个实例\
在php中闭包和匿名函数一样，都是Closure类的实例
```php
$func = function($str){echo $str;}
$func('hello world');
```
* __invoke()\
在一个对象作为函数调用时调用
```php
class A{
    __invoke($str){
        echo $str;
    }
}
$a = new A;
$a('hello world');
```
* __callStatic()\
在调用一个不存在的静态方法时被调用
```php
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
* 命名空间

### 5.4
* 数组简写
* traits

### 5.5
* yield
```php
function generator(){
    yield 1;
    yield 2;
}
foreach(generator() as $val){
    echo $val;
}
```
* list\
用于在一次操作中给一组变量赋值
```php
$array = [1,2];
list($a,$b) = $array;
```

### 5.6
* 可变参数函数
```php
function sum(...$args){
    $sum = 0;
    $sum = array_reduce($args,function($carry,$item){
        return $carry+$item;
    })
    return $sum;
}
```

### 7
* 新增标量类的参数和返回值的类型声明，如int、string、float、bool
```php
function test(int a):string{
    return "{$a}";
}
```
* null合并运算符|??
```php
$a??0;  //isset($a)?$a:0;
$a?:0;  //empty($a)?$a:0;
```
* 太空船操作符|<=>\
比较左边和右边的表达式，在小于、等于、大于时分别返回-1、0、1
```php
$a <=> $b
```
* define定义常量数组
```php
defint('ANIMALS',['dog','cat']);
echo ANIMALS[1];
```
* 匿名类
```php
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
* Unicode codepoint 转义语法
```php
echo "\u{9876}";    //输出汉字：顶
```
* Closure::call()
将一个匿名函数绑定到一个对象上闭包并执行\
```php
class Test{
    public $name = "gluge";
}
$a = function(){
    return $this->name;
}
echo $a->call(new Test);    //gluge
```
* 为反序列化unserialize()提供过滤\
可以反序列化为不完整类
```php
$data = unserialize($foo, ["allow_class"=>false]);  //反序列化为不完整类
$data = unserialize($foo, ["allow_class"=>["ClassName1","ClassName2]]); //除1和2外都反序列化为不完整类
$data = unserialize($foo, ["allow_class"=>true]);  //默认的反序列化，已经加载的类会反序列化为完整类，没有加载的类会反序列化为不完整类

$data_field = get_object_vars($data);   //可以通过这个方法获取不完整类的字段
```
* 新增IntlChar类\
用于操作unicode字符
* use导入同命名空间下的多个类、方法、常量
```php
use namespace\{class1,class2}
use function namespace\{method1,method2}
use const namespace\{const1,const2}
```
* intp()\
相除取整，返回int
```php
intp(10,3)  //3,int
float(10,3)   //3,float
```
* random_bytes(int bytes)\
按字节数生成随机字符串，实际字符串长度为bytes*2
```php
random_bytes(4);    //e62a94a2
```
* random_int(min,max)\
在区间内生成随机整数
```php
random_int(10,99)   //11
```
* preg_replace_callback_array()\
使用一个数组进行正则表达式匹配
```php
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
* 生成器有返回值，且可以在一个生成器中引入其他生成器
```php
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
* foreach 不再移动数组指针，且遍历时有更好的迭代性
```php
$arr = [0];
foreach($arr as &$v){
    $arr[1] = 1;
    echo $v;
}
//01
```
* 十六进制字符串不再认为是数字
* 数值溢出时返回null
* ini文件注释使用;而非#

### 7.4
* fn | lambda表达式
```php
fn ($x) => $x;
```

### 8
* 联合类型\
方法的参数类型和返回类型可以定义联合类型
```php
function test(int|float num):int|float{
    return num;
}
```
* 新增WeakMap\
可以创建对象到值的映射的集合，在对象存在时集合内存在映射，在对象消失时对应映射会消失\
可以理解为键为弱引用的数组
```php
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
* 新增ValueError异常类\
当传入方法的参数不符合方法的设定时会抛出
* 可以用可变参数重写方法
```php
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
* method():static\
返回方法所属的类
```php
class A{
    public function method():static{
        return $this;
    }
}
```
* obj::class 返回类型\
等同于get_class($obj)
```php
class Test{
    
}

$t = new Test;
var_dump($t::class);
```
* new、instance 可以用于表达式\
```php
class Test{}

$names = ['Test'];
$t = new ($names[0]);
```
* StringAble接口\
实现了__toString()方法的类就会被认为实现了该接口
```php
class Test{
    public function __toString(){
        return 'i am test';
    }
}
```
* trait 可以定义私有抽象方法\
使用的类需要实现方法
```php
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
```php
$a ?? throw new \Exception();
```
* 只捕获异常不存储到变量
```php
try{

}catch (\Exception){
    echo "doSomethin";
}
```
* 新增mixed类型定义，等价于array|object|bool|int|float|string|null|callable|resource
```php
public function method(mixed ...$data){

}
```
* 注解，就是c#中的特性，可以为类、函数添加元数据\
元数据可以通过反射获取
```php
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
```php
class User{
    public function __construct(public int $age, public string $name){

    }
}

$a = new User(1,'haha');
echo $a->age;
echo $a->name;
```
* match\
类似于switch
```php
$a = match(1){
    0=>'aa',
    1=>'bb',
}
echo $a;    //bb
```
* ?->\
对空安全运算符，在左不为null时按正常->运行，左侧为null直接返回null
```php
class A{
    public function getAddress(){

    }
}

$a = new A;
$phone = $a->getAddress->phone;
```
* 命名参数\
可以以参数名:值的形式传递参数，并且可以跳过默认参数
```php
array_fill(start_index:0, num:100, value: 50);
```
* Jit\
优化Opcodes被ZendVm编译为机器语言的过程

### 8.1
* enum | 枚举
```php
enum enumName{
    case Fruits = 'fruits';
    case People = 'people';
}
```


## 数组函数
* array_reduce(array,func,mixed)\
用回调函数将数组归一化为单一的值\
mixed表示在第一个数组处理前，数组的初始值
```php
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
* array_reverse(array)\
返回数组的倒序数组
* array_values(array)\
返回数组所以的值
* array_keys(array)\
返回数组所以的键
* array_flip(array)\
数组键值互换
* in_array(search,array)\
在数组中检索，返回bool
* array_search(array,search)\
在数组中检索，返回键
* array_keys_exists\
检索键名是否在数组中，返回bool
* current(array)|pos(array)\
返回当前指针对应的元素，超过长度返回false，但有可能数组当前元素就是false
* key(array)\
返回当前指针对应的键，超过长度返回null
* prev(array)\
指针前移并返回元素
* next(array)\
指针向后并返回元素
* end(array)\
指针指向最后一位并返回元素
* reset(array)\
指针指向第一位并返回元素
* each(array)\
返回当前元素键和值构建的数组，指向下个元素
* extract(array)\
把键作为变量名，把值分别赋值给键名形成的变量
```php
$a = "Original";
$my_array = array("a" => "Cat","b" => "Dog", "c" => "Horse");
extract($my_array);
echo "\$a = $a; \$b = $b; \$c = $c";
```
* compact(value1,value2,value3...)\
用给定的变量创建数组
* array_slice(array,start,end)\
从数组中取出元素，可通过后续参数配置是否保留键
* array_splice(array1,start,end,array2)\
删除array1中的对应部分，并将array2插入对应位置
* array_chunk(array,num,true|false)\
按num个分割数组，可配置是否保留键名
* array_pad(array,num,value)\
使用value将数组补到一定长度
* array_push(array,value1,value2...)\
将多个元素放入数组最后
* array_pop(array)\
取出数组第一个元素并删除
* array_shift(array)\
取出数组第一个元素并删除，数组键从0开始重写排序
* array_unshift(array,value1,value2...)\
将多个元素放入数组最前
* array_walk(array,function,arg)\
使用function遍历数组元素，arg可选
```php
function(&$value, $key, $arg){    //如果需要通过这个改变元素值，给value加引用传递
    $value = 'new value';
}
```
* array_map(function, array)\
使用function处理array并返回新数组
* array_filter(array, function)\
使用function过滤array，返回新数组
* sort(array)\
按值从小到大排序，按0重排键名
* rsort(array)\
从大到小排序，按0重排键名
* usort(array, function)\
按自定义函数排序，按0重排键名
```php
$arr = (4,2,8,6);
usort($arr, function($a, $b){
    if($a==$b){
        return $a;
    }
})
```
* asort(array)\
按值从小到大排序，不重置键名
* arsort(array)\
按值从大到小排序，不重置键名
* uasort(array)\
按自定义函数排序，不重置键名
* ksort(array)\
按键从小到大排序
* krsort(array)\
按键从大到小排序
* uksort(array)\
按自定义函数对键进行排序
* array_sum(array)\
对所有元素求和
* array_diff(arr1, arr2)\
返回数组差集，不比较键
* array_diff_assoc(arr1, arr2)\
返回数组差集，比较键和值
* array_intersect(arr1, arr2)\
返回数组交集，不比较键
* array_intersect_assoc(arr1, arr2)\
返回数组交集，比较键和值
* array_merge(arr1, arr2)\
合并数组，字符串键名相同，后面的覆盖前面的，数字键名相同，后面按需累加
* +\
合并数组，相同键名只保留后一个
* array_merge_recusive(arr1, arr2)\
递归合并，将键相同的合并到一个数组
* range(start,end)\
创建包含start~end的数组
* array_unique(arr)\
移除数组重复项，保留键名
* shuffle(arr)\
打乱数组
* array_rand(arr, int)\
随机取出n个数组元素

## 字符串函数
* strlen(string)\
获取字符长度
* strops(string, search, start)\
搜索search在string第一次出现的位置，从start开始搜索
* stiops(string, search, start)\
忽略大小写的strops
* strrops(string, search, start)\
搜索search在string最后一次出现的位置，从start开始
* submit(string, start, length)\
从start开始提取length长度字符串
* strstr(string, search)\
从0开始搜索search，返回从search位置开始到字符串结束的字符串，没有返回false
* stistr(string, search)\
忽略大小写的stistr
* strrchr(string, search)\
从string搜索最后一次出现的位置并返回search开始到字符串结束的字符串
* str_replace(search, replace, string)\
搜索search并用replace替换
* str_ireplace(search, replace, string)\
不区分大小写的str_replace
* strtr(string, search, replace)|strtr(string, [search1=>replace1...])\
和str_replace一致
* substr_replace(string, replace, start, length)\
从start开始将长度length的字符串替换为replace\
length为0时可以用作插入
* strcmp(string1, string2)\
比较字符串，>=<分别返回1、0、-1
* strcasecmp(string1, string2)\
不区分大小写的strcmp
* strnatcmp(string1, string2)\
按自然排序比较
* strnatcasecmp(string1, string2)\
不区分大小写的strnatcmp
* str_split(string, length)\
按length分割string，返回数组
* explode(search, string, int)\
按search分割string，int可选分割几次，达到int后，后续字符串不分割
* trim(string)\
移除字符串左右空格
* ltrim(string)\
移除字符串左侧空格
* rtrim(string)\
移除字符右侧空格
* chunk_split(string, int)\
每int个字符插入一个空格
* strtoupper(string)\
字符串转为大写
* strtolower(string)\
字符串转为小写
* ucfirst(string)\
大写第一个字符
* ucwords(string)\
大写最后一个字符

## 回调函数
* call_user_func\
通过间接方式调用函数的方法
```php
$a = call_user_func(function($arg){
    return "ss";
}, $arg1)


function test($a){
    return "ss";
}
$a = call_user_func("test", $arg1);

class Test{
    public function method($arg){
        return "ss";
    }
    public static function sm($arg){
        return "ss"
    }
}

$t = new Test();
$a = call_user_func([$t,'method'], $arg);
$a = call_user_func(["Test","sm"], $arg);
```
* call_user_func_array\
与call_user_func类似，唯一区别只有参数需要用数组
```php
class Test{
    public function method($arg){
        return "ss";
    }
    public static function sm($arg){
        return "ss"
    }
}

$t = new Test();
$a = call_user_func_array([$t,'method'], [$arg]);
$a = call_user_func_arrry(["Test","sm"], [$arg]);
```

## '|"
单引号效率更高\
从Opcodes编译而言，双引号会将变量存储在临时变量中，然后将字符串写入，再用变量替换字符串，而单引号没有这个过程，因此单引号效率更高

## for|foreach
foreach效率更高\
从语言结构而言，foreach是指针下移的过程，而for需要每次比较i值的大小

## 魔术方法
* __construct()\
构造函数，类实例化时就会调用
* __destruct()\
析构函数，在实例被销毁时调用
* __clone()\
在实例被clone时调用
```php
$a = new Test();
clone($a);
```
* __toString()\
当对象被当成字符串使用时自动调用\
与C#不同的是php基类没有实现toString
* __invoke()\
当对象被当成函数使用时自动调用
```php
class Test(){
    public function __invoke(){
        echo "我被当成函数使用了";
    }
}

$a = new Test();
$a();
```
* __call(string functionName, array args)\
在实例上调用不存在的方法时调用
* __callStatic(string functionName, array args)\
在类上调用不存在静态方法时调用
* __get(string fieldName)\
访问对象protect或private字段时自动调用\
类似C#的属性的get，但针对所有目标字段
```php
class Test{
    private $name;
    private $age;

    public function __construct(string $name, int $age){
        $this->name = $name;
        $this->age = $age;
    }

    public function __get(string $fieldName){
        if($fieldName == 'age'){
            return $this->age - 10;
        }else{
            return $this->$fieldName;
        }
    }
}

$a = new Test('zz', 50);
echo $a->age;   //40    这里实际直接调用__get方法，因此可以获得私有变量
echo $a->name;  //zz
```
* __set(string fieldName, value)\
设置对象protect或private字段时自动调用\
类似C#属性的set，但针对所有目标字段
* __isset(string fieldName)\
对对象protect或private字段使用isset或empty时自动调用
* __unset(string fieldName)\
对对象protect或private字段使用unset时自动调用
* __sleep()\
对对象进行序列化时自动调用，在序列化前执行，返回需要序列化的字段名称的数组\
类似于C#实现ISerializabe接口的GetObjectData方法
```php
class Test(){
    public $name;
    public $age;
    public $time;
    public function __sleep(){
        $this->name = "zz";
        $this->age = 10;
        return ["name", "age"];     //执行序列化时只序列化name和age
    }
}
```
* __weakup()\
当执行反序列化为对象时会调用\
类似C#反序列化时的特殊构造器或应用了OnDeserialiedAttribute特性的方法
* __autoload()\
尝试加载未定义的类\
定义__autoload后，如果php执行过程中发现未定义的类，就会自动执行__autoload尝试加载
```php
function  __autoload($className) {
    $filePath = "project/class/{$className}.php";
    if (is_readable($filePath)) {
        require($filePath);
    }  
}  

if (条件A) {
    $a = new A();
    $b = new B();
    $c = new C();
} else if (条件B) {
    $a = new A();
    $b = new B();
}
```
* __debugInfo()\
当对对象执行var_dump时自动调用
* __set_state(array fields)\
当对对象使用var_export方法时自动调用，返回一个类的实例
```php
class Test(){
    public $name;

    public function __set_state($fields){
        $this->name = "zz";
        return $this;
    }
}
```

## 魔术常量
* __LINE__
返回所在行数
* __FILE__
返回当前文件的绝对路径
```
c:\www\1.php
```
* __DIR__
返回当前目录的绝对路径
```
c:\www
```
* __FUNCTION__
返回函数名或空字符串
```
getList
```
* __NAMESPACE__
返回命名空间
```
app\controller
```
* __CLASS__
返回类名包含命名空间
```
app\controller\IndexController
```
* __METHOD__
返回方法名包含类和命名空间
```
app\controller\IndexController::index
```
* __TRAIT__
返回trait名包含命名空间
```
app\trait\MyTrait
```



# composer
* --ignore-platform-reqs
忽略php版本匹配进行包安装，对某些写明php版本但实际在范围外仍可使用的包可以使用这个进行安装



# 依赖注入
> **依赖注入**
> 把有依赖关系的类放到容器中，解析出的实例注入类中，这个过程就是依赖注入
> 
可以方便的切换注入的内容

## 构造函数注入
从构造器注入
```php
class test{

    private $class;

    public function __construct(class $class){
        $this->class = $class;
    }
}
```

## setter注入
编写特定的set方法注入
```php
class test{

    private $class;

    public function setClass(class $class){
        $this->class = $class;
    }
}
```

## 接口注入
类实现接口，接口中某方法参数就是需要注入的内容\
由于实现接口功能反而需要类中扩展很多代码，因此不建议使用



# PHP 接口版本控制
> 待施工



# PHP 慢sql查询
> 待施工



# PHP RPC的实现

## RPC和消息中间件
RPC是同步调用的过程，业务直接存在耦合，业务A请求业务B，业务B失败会导致业务A也失败\
消息中间件是异步调用的过程，业务A负责把消息推入队列中，业务B订阅队列并消费，业务A在推入队列完成时就已经完成了自己的业务，可以返回成功，达到了业务之间的解耦\

## 原生实现
```php
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
可以实现RPC传输的包
```php
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
```php
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



# 接口鉴权与JWT

## 验证
验证就是核实用户身份，就是登录的过程

## 受权
基于的状态给用户一些权力，一般通过cookie、session、token、OAuth

## cookie
存储在客户端的数据，方便客户端存储一些数据以便再下次请求时提交\
cookie存储的内容不能超过4k，但可以长久储存

## session
存储在服务端的数据，用于指向对应的用户\
session存储的内容无大小限制，但一般只能短时间内存储\
缺点：分布式系统的seesion共享

## token
登录后获取token，一般由用户id、时间戳等加盐加密而得，客户端凭token进行接口鉴权，服务器端根据token从redis等缓存或数据库中获取信息\
缺点：服务端需要存储token，每次访问redis也是开销\
缺点：服务器可能还需要读取数据库以补全token未包含的信息

## JWT
是token的一种延申或优化，相当于同时传输了token及相关信息\
一般客户端从认证服务器登录，获取JWT后，请求应用服务器使用，实现了单点登录

### JWT构成情况
xxx.yyy.zzz格式
* Header\
一般包含alg加密算法和typ令牌类型两部分，然后将这个json用base64转换为字符串
```json
{
    "alg":"HS256",
    "typ":"JWT"
}
```
* Payload\
载体部分，默认有包括iss(发行人)、sub(主题)、aud(用户)、exp(过期时间)、nbf(生效时间)、iat(签发时间)、jti(JWTid)，也可以存储自定义字段，比如ip、机器码等，将这个json用base64转为字符串保存
```json
{
    "sub":"我就是GUNDAM",
    "aud":1,
    //自定义字段
    "ip":"192.168.0.1"
}
```
* Signature\
将header和payload字符串加上secret用header中的加密方式进行加盐加密\
应用服务器一般通过前两段数据经过同一个算法计算是否可以得出相同的签名

### JWT优缺点
* 优点\
json通用性高\
payload中可以保存一些信息，减少数据库查询次数\
服务器不用保存信息
* 缺点\
JWT签发后无法改变JWT的权限，签发后在有效期内始终保持有效状态\
JWT续签只能签发新的JWT
* 缺点解决方案\
引入Redis进行JWT控制



# Laravel

## ENV
一般用于存储因环境不同而改变的配置信息\
在生命周期启动时加载到$_ENV中
* env(envName, defaultValue)
```php
env('app_debug', false);
```

## config
用于存储固定配置，一般搭配env使用\
* config(fileName.configName, defaultValue)
返回config值
```php
config('app.timezone', 'Asia/Shanghai');
```
* config([fileName.configName => setValue])
设置config值
```php
config(['app.timezone' => 'Asiz/Shanghai']);
```
* artisan config cache
可以将config配置合并成一个缓存文件，可以提应用速度

## 目录结构
1. app | 应用的核心代码，代码由composer自动加载
   1. Broadcasting | 广播，通过make:channel生成
   2. Console | artisan命令，通过make:command生成
   3. Events | 事件，类似于观察者，event:generate、make:event生成
   4. Exceptions | 异常处理，可通过修改Handler类自定义异常呈现方式
   5. Http | 应用逻辑
   6. Jobs | 队列，通过make:job生成
   7. Listeners | 处理event类，通过event:generaate、make:listener生成
   8. Mail | 电子邮件，通过make:mail生成
   9.  Models | Eloquent orm模型类
   10. Notifications | 消息通知，通过make:notification生成
   11. Policies | 授权策略，通过make:pollicy生成
   12. Providers | 服务提供者，引导应用程序处理请求
   13. Rules | 验证规则，通过make:rule生成
2.  bootstrap | 启动框架的app.php文件
    1.  cache | 用于优化框架性能的核心缓存文件
3.  config | 配置文件
4.  database | 数据库相关文件
    1.  factories | 模型工厂
    2.  migrations | 数据库迁移文件
    3.  seeders | 种子生成文件
5.  lang | 语言文件
6.  public | 包含index.php，是所有进入应用的请求的入口文件，还包含其他被公共访问的资源文件
7.  resources | 包含views及未编译的资源文件
8.  routes | 路由定义文件
    1.  web.php | 提供csrf保护及cookie加密，一般用于http会话
    2.  api.php | 旨在存放需要通过令牌验证身份的接口路由
    3.  console.php | 定义控制台命令
    4.  channels | 注册事件广播
9.  storage | 存储其他缓存
    1.  app | 存储应用生成的任何文件
    2.  framework | 存储框架缓存
    3.  logs | 日志文件
10. tests | 单元测试
11. vendor | composer依赖项

## 生命周期
> 简单归纳
> 创建应用实例$\rightarrow$绑定内核$\rightarrow$注册服务提供者$\rightarrow$传递请求至引导程序
1. 从web服务器应用定向到index.php入口文件
2. composer加载所有依赖项
3. laravel应用|服务容器实例创建
4. 绑定Http或Console内核，后续步骤以Http内核为例
5. Http内核基础Kernel（翻译为内核）类，定义了一系列处理请求前应先运行的bootstrappers数组，这些都是引导程序，配置异常处理、日志、应用环境检测等任务
6. 注册服务提供者，在config/app.php中定义，调用register方法，再调用boot方法，提供了如数据库、队列、验证、路由等组件
7. 将请求发送至路由组件解析，Http中间件处理请求
8. 路由或控制器返回的响应由Http中间件的handle方法返回
9. 通过send方法将响应发送

## 服务容器
依赖注入提供了一种便利的改变依赖项的方式，服务容器则是用以执行依赖注入的工具\
可以理解为，laravel应用都是在服务容器中执行的，当需要依赖注入时，会在注入前经过服务容器的调节，达到自动注入或变更注入项的功能

> **核心概念 - 绑定**
> 服务容器的核心概念是绑定，通过将需要注入的项绑定到容器，可以在未来需要注入时由服务容器自动注入或手动解析
>

### 绑定到服务容器
一般在服务提供者的register方法中定义
* bind
最普通的绑定方式，注入时提供一个类实例的闭包
```php
$this->app->bind(ClassName, fn($app) => new ClassName());
```
* singleton
单例绑定，单次进程中保证存活
```php
$this->app->singleton(ClassName, fn($app) => new ClassName());
```
* scoped
单例绑定，在开始新生命周期时刷新
```php
$this->app->scoped(ClassName, fn($app) => new ClassName());
```
* 接口绑定
绑定接口实现，可以给一个接口绑定对应的注入类
```php
$this->app->bind(InterfaceName, ClassName);
```
```php
class Test{
    //此时ClassName会被注入
    public function __contruct(InterfaceName $class){
        
    }
}
```
* 上下文绑定
当不同的类绑定同一个接口注入时，可以通过此绑定不同的注入实例
```php
$this->app->when([ClassName1])
          ->needs(NeedInterfaceName)
          ->give(function(){
            return new NeedClassName1();
          });
$this->app->when([ClassName2])
          ->needs(NeedInterfaceName)
          ->give(function(){
            return new NeedClassName2();
          });
```
* 原语绑定
在需要注入参数时使用
```php
$this->app->when([ClassName1])
          ->needs('$ArgName')
          ->give(Value);
$this->app->when([ClassName1])
          ->needs('$ArgName')
          ->giveTagged(TagName);
$this->app->when([ClassName1])
          ->needs('$ArgName')
          ->giveConfig(FileName.ConfigName);
```
* 变长参数绑定
```php
$this->app->when([ClassName])
          ->needs(NeedInterfaceName)
          ->give(fn ($app) => [
            $app->make(NeedClassName1),     //注册时解析
            NeedClassName2      //注册时不解析
          ]);
```
* 标签
将已经绑定的类别归纳为一个组
```php
$this->app->tag([BindedClass1,...], TagName);
$this->app->when([ClassName])
          ->needs(NeedInterfaceName)
          ->giveTagged(TagName);
```
* 继承绑定
使用子类的实例更改已经绑定的父类实例\
```php
$this->app->extend(ClassName, fn ($app) => new ChildClassName);
```

### 解析绑定的内容
* make
```php
$this->app->make(ClassName);
```
* makeWith
传入参数的解析
```php
$this->app->makeWith(ClassName, ['arg1' => value1...]);
```

### 实例的方法调用
可以解析并调用实例的方法
```php
App::call([new ClassName(), MethodName]);
```

### 容器事件
* resolving
每次解析实例都会调用一个相应的事件，可以通过此方法开始监听
```php
//解析任何对象时都会调用
$this->app->resolving(function($object, $app){

});
//解析特定类型对象时调用
$this->app->resolving(ClassName, function($className, $app){

})
```

## Provider | 服务提供者
为laravel提供服务\
简单来说就是在provider中注册了需要的服务

### 创建Provider
```sh
php artisan make:provider ProviderName
```
实际创建了文件并将其注册到app.providers配置中

### 配置Provider
```php
class MyProvider extends ServiceProvider implements DeferrableProvider{

    $binds = [

    ];

    $singletons = [

    ];

    public function register(){

    }

    public function boot(){

    }
}
```
* register
provider加载时执行的方法，一般只在此注册提供的服务，因为有可能无法访问到其它服务提供者
* boot
所有provider加载完毕后执行的方法，可以访问其它provider提供的服务
* $bindings
需要绑定的服务数组
* $singletons
需要绑定的单例数组
* DeferrableProvider
惰性加载的Provider，只有当需要访问Provider中提供的服务时才加载

## Facades
提供了一种模拟静态方法访问实例方法的方式，将实例模拟成帮助类\
实际是通过__callStatic方法提供相关方法的访问

## laravel中间件
只用于Http内核的中间件，在Http的Kernel.php文件中注册
* 全局中间件
  每个Http请求中都会运行的中间件
* 组中间件
  按路由文件的名称分组的中间件，会自动运行于对应的路由
* 路由中间件
  为中间件提供键，以便在路由中使用middleware方法添加

### 实现原理
使用array_reduce返回中间件执行的嵌套闭包\
由于array_reduce返回的闭包的最外层是传入array的最后一个元素，而最外层闭包又是最先执行的，因此先通过array_reverse将中间件数组倒序，以获得正序执行的嵌套闭包

## 渲染引擎
blade模板引擎



# hyperf
高性能、灵活性的渐进式的php协程框架
* 高性能\
基于swoole协程保证高性能输出
* 灵活性\
基于PSR和hyperf定义的契约，使大部分组件或类可以替换，类似于类的继承或接口的继承
* 渐进式\
没有强主张，提供了很多功能，但都不是必须使用的\
* hyperf的协程\
只有单线程在运行的多线程处理方式，线程遇到io就挂起等待，其他线程执行\
将同步等待io的时间用于其他线程的处理，因此对于io较多的服务应用有巨大提升，而对于计算算密集型的项目，提升只体现在线程创建的资源消耗上

## php-fpm|hyperf性能差异的原因
主要还是进程与线程的区别，进程可以理解为是排他的，线程则在一定程度上是资源共享的\
php-fpm对于每个请求建立一个进程，进程内同步执行所有内容\
hyperf是以task的方式创建线程，异步处理io内容
* 进程与线程创建所消耗资源是不一样的
* hyperf有很多地方使用了池模式，比如mysql的连接池(池模式可通过线程池理解)



# 微服务
协同工作的小而自治的服务

## 优点
* 业务解耦\
使业务逻辑更清晰，迭代效率更高，缩小业务更改带来的影响
* 性能提升
  1. 通过rpc代替http降低传输成本\
   同样的tcp数据包，rpc头部比http更小，rpc的字节流传递数据所在整个传输量的比例更高\
   在业务互相依赖时使用
  2. 通过消息中间件进行消息传输
   将业务进一步解耦\
   在业务无依赖时使用
* 弹性重构\
针对对不同服务的功能、承载量进行弹性设置，通过限流、熔断等方式降低单服务问题引起的系统瘫痪

## 缺点
* 需要较强的运维能力
* 出现数据混乱、内存溢出和协程问题

## 服务治理
微服务的服务治理，是一个应用注册、管理、使用的过程，相当于k8s对容器的管理，以下以consul为例
* 服务注册\
服务启动时在服务中心进行注册，提供ip、port等信息
* 服务健康检查\
服务中心通过心跳方式定期检查服务状态，如果服务不健康就从其他相同服务的不同节点提供服务
* 服务重试\
设置重试次数和熔断机制，避免多次请求造成的服务器资源浪费
* 服务熔断和降级\
当a服务请求b服务导致b服务出现问题，在a为次要服务，b为核心服务的情况下，为保证b服务正常提供服务，在a建立后备措施，即当a请求b超时或其他情况达到一定阈值时，a启动降级方法
* 服务限流\
在高并发场景下，对服务进行限流，只开放固定时间内固定访问数量的服务访问，或将每次请求的服务消耗作为参数，对该参数求和进行限流
* 调用链追踪\
为每个请求建立一个id，每次日志建立都带上该id，进行日志访问追踪
* 服务监控\
通过可视化工具监控服务消耗的服务器资源情况，cpu、内存等
* 自动化运维\
一般可以通过k8s实现