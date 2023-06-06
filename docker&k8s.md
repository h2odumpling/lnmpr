# docker
应用容器引擎，将应用及依赖打包在一个可移植镜像中，可以发布到任何linux或windows操作系统上\

## docker容器
将应用程序及其依赖打包在一起创建的抽象层\

## dockerfile
用于构建从无到有的镜像的文件\

* FROM
设置使用的基础镜像\
```
# FROM <image>:<tag|digest>
FROM mysql:5.7 
```
* MANAGER
作者信息
```
# MANAGER <name>
MANAGER h2odumpling h2odumpling@vip.qq.com
```
* RUN
编译镜像时运行的脚本
```
# RUN <command>
RUN apk update
```
* CMD
运行镜像容器时运行的脚本
```
# CMD <param1> <param2>... 当设置ENTRYPOINT的情况下会直接调用ENTRYPOINT
# CMD <command> <param1> <param2>...
CMD echo "this is a test"
```
* LABEL
设置镜像的标签或元数据\
```
# LABEL <KEY>=<VALUE> <KEY>=<VALUE>...
LABEL version="1.0.0.0" description="这是一个介绍"
```
* EXPOESE
镜像容器暴露的端口号\
```
# EXPOESE <port1> <port2>
EXPOESE 80 433 11211/tcp 22122/udp
```
* ENV
设置容器的环境变量\
```
# ENV <key> <value>
ENV myName h2odumpling
```
* ADD
编译时复制文件到容器中，会自动解压缩，可支持网络资源，不解压网络资源\
```
# ADD <src> <dest>
# ADD ["<src>","<dest>"] 如果路径含有空格
ADD hom* /mydir/
ADD hom?.txt /mydir/
```
* COPY
编译时复制文件到容器中，不支持网络资源\
* ENTRYPOINT
设置容器的入口\
```
# ENTRYPOINT <command> <param1> <param2>...
# ENTRYPOINT ["<filename>","<param1>","<param2>"...] 可执行文件
ENTRYPOINT ["nginx"]
```
* VOLUME
设置容器挂载卷\
```
# VOLUME ["path1","path2"...]
VOLUME ["var/www"]
```
* USER
运行容器时的用户名\
```
# USER <user>
# USER <user>:<group>
# USER <uid>:<gid>
USER www
```
* WORKDIR
设置RUN、CMD、ADD、COPY指令的工作目录\
```
# WORKDIR <path>
WORKDIR /a
WORKDIR b   //此时为/a/b
```
* ARG
设置编译镜像时加入的参数\
```
# ARG <name>
# ARG <name>=<value>
ARG site=zzz
```
* ONBUILD
设置镜像的ONBUILD指令，当镜像被其他镜像作为基础镜像时会触发\
```
# ONBUILD <command>
ONBUILD ADD . /app/src
```
* STOPSIGNAL
设置容器的退出信号\

### docker build
使用某目录的dockerfile构建镜像\
```
# docker build -t <image_name>:<version> <path>
docker build -t test:1.0.0 .    //.表示当前目录
```
* -t 指定镜像名称
* -f 指定dockerfile文件，不指定则默认使用指定目录下的名为Dockerfile的文件构筑镜像

## docker-compose
是一个命令行工具，可以定义和运行多容器docker应用程序\
一个依赖于docker用以编排多容器的工具\
主要运用于有多个镜像时，配置各个容器的各类参数，比如mysql、php、nginx的联合启动，而不用docker则需要针对每个容器进行一次启动\

### yaml文件
* version
指定dockercompose文件版本，常见有1、2、3
* service
指定要添加的容器服务，可以从现有的镜像指定，也可以通过dockerfile文件直接打包一个镜像容器\
```
services:
    redis:
        image: redis
        ports:      //指定端口同时映射到随机临时端口
            - "6379"
        network
            - frontend
        depends_on:     //指定服务启动、停止先后顺序及依赖关系，自动启动依赖但未启动的容器
            - webapp
        depoly:     //指定部署和运行服务的相关配置
            replicas: 6
            placement:
                max_replicas_per_node: 1
            endpoint_mode: vip  //vip表示docker为服务配置前端虚拟ip，客户端通过虚拟ip连接此服务，docker在客户端和可用工作节点直接路由请求，不关心节点负载；dnssr表示启用DNS轮询，客户端通过轮询方式连接到其中之一
            labels:
                version:"3.1.1"   //指定元数据
            mode: global     //指定容器的副本模式，global每个swarm节点只有一个该服务器，replicated集群中存在指定份数的服务容器副本
            replicas: 6     //指定容器副本数
            placement:      
                constraints:    //constraints指定运行该容器符合的节点
                    - "node.role=manager"
                preferences:   //preferences指定容器的分配策略
                    - spread: node.labels.zone
            resources:      //配置资源限制
                limits:     //上限
                    cpu: "0.50"
                    memory: 50M
                reservations:    //下限
                    cpu: "0.25"
                    memory: 25M
            restart_policy:     //容器重启策略
                condition: on-failure   //none、on-failure、any     什么情况下重启
                delay: 6s   //尝试重启的等待时间
                max_attempts: 3     //重启尝试次数
                window: 120s    //决定重启是否成功的等待时间
            rollback_config:    //配置更新失败时如何回滚服务
            update_config:  //配置如何更新服务
                parallelism: 2  //一次更新的容器数量
                delay: 5s   //更新容器等待时间
                order: stop-first     //更新顺序，stop-first在开启新任务前停止旧服务，start-first首先启动新任务
            env_file:   //从文件中获取环境变量
                - ./common.env
                - ./web.env
            enviroment:     //设置环境变量
                SHOW: 'true'
                RACK_ENV: development
            expose:     //指定暴露的端口，只能被连接的服务访问
                - "3000"
        link:   //遗留功能，建议使用network代替
            - "db"
        logging:    //日志配置
            driver: syslog  //可以选择json-file、syslog、none等
            options:    //为日志驱动指定记录选项
                syslog-address: "tcp://192.168.0.42:11211"
                max-size: "200k"
                max-file: "10"
        network_mode: "bridge"  //指定使用服务或容器的网络，bridge、host、none、service:[service_name]、container:[container_name]
        networks:   //指定要加入的网络
            - network1
        restart: always   //重启策略，always在退出时重启容器，on-failure在以非0状态码退出容器时重启，unless-stoped在容器退出时重启，但不考虑docker守护进程启动时已经停止的容器
        stop_signal: CUSTOMSIGNAL   //设置停止容器的的信号，默认是SIGTERM
        tmpfs:      //挂载临时文件夹到容器
            - /run
        ulimits:    //指定容器的最大进程数，或设置一个范围
            nproc: 65535
            nofile:
                soft: 20000
                hard: 40000
        volumes:    //指定挂载的数据卷名称或路径，path:path或volume_name:path
            - /var/lib/mysql
            - datavolume
    webapp:
        build:
            context: ./dir  //指定目录
            dockerfile: Dockerfile  //指定dockerfile文件
            args:   //指定build参数
                - buildno=1
                - otherparam    //不指定值的情况下会从当前compose环境中寻找对应参数取值
            cache_from:     //指定缓存解析镜像列表
                - alpine:latest
                - corp/web:3.14
            labels:
                - "version=3.1.1"   //指定元数据
            network: custom_network   //设置容器网络连接，设为none则禁用网络
            shm_size: 1000000   //按字节数指定容器大小或者表示字节的字符串如"2gb"
            target: prod    //指定构建的阶段到某阶段为止
            cap_add:    //添加容器内核能力
                - ALL
            cap_drop:   //删除容器内核能力
                - NET_ADMIN
            cgroup_parent: customed     //指定可选的父控制组
            command：bundle exec thin -p 3000   //覆盖容器启动时默认执行的命令
            configs:
                - my_config     //授予容器访问config的权限并挂载到对应的config_name上
                - source: other_config    //docker中已经存在的config名称
                  target: /path     //指定要挂载到此容器的路径及文件名
                  uid: '100'    //设置该文件拥有者的uid
                  gid: '100'    //设置该文件拥有者的gid
                  mode: 0440    //以八进制表示文件权限，0444可读、0440用户群可读
```
* configs
```
configs:
    my_config:
        file ./my_config.txt
    other_config:
        external: true  //设置使用已经通过docker config create命令或已经在其他栈堆部署定义
```
* networks
网络配置\
```
networks:
    network1:
        driver: bridge    //网络所用的驱动，单主机一般用bridge桥接，多主机用overlay驱动一个多节点间的命名网络
        driver_opts:    //网络驱动选项
            type: "custom"
        attachable: true    //指定服务外的容器是否可以连接到此网络
        name: mynam     //设置自定义名称
```
* volumes
数据卷配置\
```
volumes:
    test:
        driver: local   //数据卷使用的驱动
        driver_opts:    //数据卷驱动选项
            type: "nfs"
        external: true     //指定数据卷是否是外部数据卷
        name: myname    //设置自定义名称
```
* 扩展字段
以x-开头命名，定义可以在配置文件中重用的的配置片段\
```
x-custom:
    &default-logging
    options:
        max-size: '12m'
        min-size: '5m'
```
```
services:
    web:
        image:web
        logging: *default-logging
```



# k8s|kubernetes
由google开发的类似于docker swaram的应用，用于管理多主机上的容器应用\

## 主要功能
* Pod：多进程协调工作
* 存储系统挂载
* 应用状态监控
* 应用实例复制
* Pod伸缩扩展
* 负载均衡
* 滚动更新
* 资源监控
* 日志访问
* 调试应用程序
* 提供认证和授权

## k8s组件

### Master
提供集群管理控制中心\
可以在任意节点运行，但一般在单独一台机器上运行Master且不在此机器上运行其他容器应用\

* kube-apiserver
用于暴露k8s接口，任何操作或请求都通过kube-apiserver提供的接口进行\
* ETCD
默认存储系统，保存所有集群数据，可用于备份\
* kube-controller-manager
运行管理控制器，处理常规任务的后台线程\
包括Node（维护节点）、Replication（维护pod）、Endpoints、Service Account和Token的控制器\
* cloud-controller-manager
云控制器管理器负责与底层云提供商的平台交互\
包括Node、Route(路由)、Service、Volume控制器\
* kube-scheduler
监视新创建没有分配到Node上的Pod，并为Pod选择一个Node\
* addons
插件用以实现Pod和Services功能\
* DNS
集群DNS，为Kubernetes services提供DNS记录\
* kube-ui
用户界面，提供了集群状态的基础信息\
* 容器资源监控
提供监控数据的ui浏览\
* Cluster-level Logging
日志相关的服务，保存或搜索日志\

### Node
节点组件，提供k8s的运行时环境，维护Pod\
* kublet
节点代理，监视分配给Node的Pod\
安装Pod所需的Volume\
下载Pod的Secrets\
Pod中运行Docker或其他容器\
定期检查容器健康\
报告Pod状态并在必要时创建一个镜像\
报告Node的状态\
* kube-proxy
维护网络规则，执行连接转发实现k8s服务抽象\
* docker
用于运行容器\
* RKT
docker工具的替代方案
* supervisord
轻量级监控系统，保证kublet和docker的运行\
* fluentd
守护进程，可提供cluster-level-logging\

## k8s Object|k8s 对象
k8s系统中的持久实体\
* 容器化应用
* 应用使用的资源
* 应用运行的策略
k8s对象可以通过k8s api来调用，可以通过CLI或自己的程序，但自己的程序现在只有golang客户端库\
可通过编写yml文件（类似于dockerfile）来创建\

### k8s file
* apiVersion
创建k8s对象的k8s api版本\
* kind
创建什么样的对象\
* metadata
对象的唯一标识数据，包括name、uid、namespace\                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        x
name：一个对象拥有一个name，在对象被删除后可以通过name建立新对象，可以用于资源引用的url中\
uids：由k8s生成的唯一性uid，在每次创建对象时生成\
