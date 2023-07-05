# 1**
消息类型的状态码

* 100|Continue\
表示服务端已经接收部分请求内容，需要客户端继续发送剩余内容
* 101|Switching Protocols\
服务端理解了客户端请求，但需要客户端更换协议来完成请求\
一般在新版http协议比旧版更有优势，或切换到一个实时且同步的协议来利用协议的特性时
* 102|Processing\
表示将继续处理

# 2**
代表请求已经成功，且已经完成

* 200|OK\
请求成功
* 201|Created\
请求成功，并且有新资源被建立
* 202|Accepted\
请求已经接收，但还未处理，一般用于异步处理的时候
* 203|Non-Authoritative Information\
服务端处理了请求，但返回的实体头部不是原始服务端的有效数据
* 204|No Content\
请求成功，但没有返回body，没有任何消息信息
* 205|Reset Content\
与204一致，但要求重置视图，一般用于表单提交后重置表单以开始下一次输入
* 206|Partial Content\
成功响应GET请求，一般用于大文件下载时的多下载段下载\
该请求头必须包括Content-range来表示本次响应中返回的文件内容
* 207|Multi-Status\
代表消息是一个xml格式的消息，并且可能包含一系列独立的响应代码

# 3**
重定向有关的状态码，表示客户端需要采取重定向处理来完成请求\
当后续请求是GET或HEAD时，浏览器会自动进行重定向操作，且重定向不能超过5次

* 300|Multiple Choices\
有多个可以选择的重定向信息，用户或浏览器选择其中最优的进行处理即可
* 301|Moved Permanently\
资源已经被永久移动到新位置
* 302|Move Temporaily\
请求的资源从别的URI被响应
* 303|See Other\
当前请求可以在别的连接被找到，且应该使用GET方法
* 304|Not Modified\
客户端将同个请求在不同时间点发送，当客户端存有缓存且服务端返回数据没有更新时，应返回304表示读取缓存即可
* 305|Use Proxy\
被请求资源需要通过指定代理才能被访问
* 306\
已废弃
* 307|Temporary Redirect\
请求的资源临时从别的URI被响应

# 4**
请求错误，一般是客户端发起的请求有问题导致服务端无法处理

* 400|Bad Request\
请求无法被服务端理解\
一般是客户端请求头有问题
* 401|Unauthorized\
当前请求需要用户验证
* 402|Payment Required\
为需求而预留，暂未使用
* 403|Forbidden\
服务端理解请求，但拒绝响应\
一般是ip地址黑名单或web服务端设置问题
* 404|Not Found\
服务端理解请求，但资源未找到导致请求失败\
客户端写错地址
* 405|Method Not Allowed\
客户端的请求方法不能被用于请求当前资源\
客户端写错请求方式
* 406|Not Acceptable\
请求内容不满足请求头条件，无法被服务端理解
* 407|Proxy Authentication Required\
与401类似，但必须在代理服务器上进行身份验证
* 408|Request Timeout\
客户端没有在服务端等待时间内发送请求
* 409|Conflict\
请求资源与当前存在冲突\
一般当客户端使用PUT请求处理的时候，但此时数据已经被第三方修改，导致冲突
* 410|Gone\
请求资源在服务器上已经永久不可用
* 411|Length Required\
客户端需要定义Content-Length
* 412|Precondition Failed\
客户端在请求头中的信息验证失败
* 413|Request Entity Too Large\
客户端发送的请求大小超出了服务端处理的范围
* 414|Request-URI Too Long\
客户端请求的URI长度过长，一般发生在POST使用GET请求导致表单内容变成了URI字符串中的一部分
* 415|Unsupported Media Type\
客户端请求的资源，不是服务端支持的格式
* 416|Requested Range Not Satisfiable\
客户端请求的Range超过当前服务端资源
* 417|Expectation Failed\
请求数据在当前服务器无法被满足
* 421|Misdirected Request\
请求指向无法响应的服务器
* 422|Unprocessable Entity\
请求格式正确但语义错误\
常被用于接口参数是json格式但参数错误
* 423|Locked\
当前资源被锁定
* 424|Failed Dependency\
由于之前某个请求的错误导致当前请求错误
* 425|Too Early\
服务器不愿意执行请求，因为有被攻击的风险，例如重放攻击
* 426|Upgrade Required\
客户端应当切换到TLS/1.0
* 449|Retry With\
请求应该在适当的操作后重试
* 451|Unavailable For Legal Reasons\
请求因法律原因无法响应

# 5**|6**
服务器错误

* 500|Internal Server Error\
遇到一个无法处理的错误，一般是源代码报错
* 501|Not Implemented\
服务端不支持当前请求所要求的功能\
一般是无法识别请求的方法
* 502|Bad Gateway\
作为网关代理从上游服务器获得无效相应\
在php的情况下，一般是php-fpm没有正常启动，或phpini设置的执行时间过短而处理时间过长导致超时
* 503|Service Unavailable\
服务器过载或维护状态下，不想接受客户端请求而拒绝
* 504|Gateway Timeout\
作为网关代理未能及时从上游服务器获得响应\
在php的情况下，一般是nginx等web服务器设置的响应时间过短，而php-fpm的处理时间过长
* 505|Version Not Supported\
服务端不支持当前的http协议
* 506|Variant Also Negotiates\
服务器内部配置错误
* 507|Insufficient Storage\
服务器无法存储完成请求所需的内容
* 509|Bandwidth Limit Exceeded\
服务器达到带宽限制
* 510|Not Extended\
获取资源所需的策略没有被满足
* 600|Unparseable Response Headers\
没有返回响应头，只返回响应内容