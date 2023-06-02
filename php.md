# php基础

## require|require_once|include|include_once|use
* include 遇到文件错误产生一个警告但程序继续运行
* include_once 与include一样，但在代码已经被导入情况下不会继续导入
* require 遇到文件错误直接停止程序运行
* require_once 与require一样，但在代码已经被包括情况下不会继续包括
* use 按命名空间和类名导入



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