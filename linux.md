# 参数
cmd [options] [arg]\
无参option可以多个连用，且位置可以任意调换\
```
ls -al
```
当有参option和无参options连用时，有参option只能放于最后，且只能使用一个，且之后紧跟所需的参数\
```
tar -zxf filename
```



# 管道
可以将多个命令使用|连用，前一个命令的输出变为后一个命令的输入，最后输出结果\
a|b 即为a的结果作为b的输入，输出b的结果\
```
ls -a | grep t
```

* xargs
通过管道把前一个命令的输出当作参数传递给下一个命令\
其实相当于是一种回调函数\
```
cat url.text | xargs weget -c
```



# 常用命令

## 文件管理
* cd
打开目录\
* ls
查看当前目录下所有目录\
-a 查看文件及目录\
-l 列出详细信息\
* echo
输出信息\
* cat
打印文件到命令行\
* chown
改变文件所属用户\
* chgrp
改变文件所属用户组\
* chmod
改变文件读写权限\
文件权限分为读4、写2、执行1\
显示为[-|l|d]rwxr--rw-\
第一位的-表示普通文件、l表示链接文件、d表示目录\
第一个rwx表示所属用户的权限\
第二个rwx表示所属用户组的用户的权限\
第三个rwx表示其他用户的权限\
数字则为[7|6|5|4|3|2|1|0]~3
* wget
下载文件\
* grep
正则查找\
* wc
统计文件行数、字数、字符数\
* more
显示文件\
加载整个文件，然后将文件显示满屏幕，可通过空格或b进行翻页或退回，退出后more显示的内容仍然存留在shell上\
* less
显示文件\
只加载部分文件，可按上下键查看其他内容，退出后less显示的内容不会存留在shell上\
* find [dir] [options]
查找文件\
* touch
文件存在时会改变访问时间和修改时间，文件不存在就新建文件\
* cp
复制文件\
* mv
移动或重命名文件\
* rm
删除文件\
-r 递归删除文件夹，删除子目录及文件
* rmdir
删除空文件夹\
* tree
树形结构显示目录\
* pwd
显示当前目录路径\
* ln
创建链接文件\
* head|tail
显示文件的头或尾\
* sed
流编辑器，可以对文件进行修改操作\



## vi|vim 文件编辑
有命令、编辑、底行三种模式，默认进入命令模式，在命令模式中i可以进入插入模式、:可以进入底行模式，插入模式和底行模式只能Esc回到命令模式\
* w
表示保存
* q
表示退出
* q!
不保存退出\
* set number
显示行号\
* /|?
查找关键字\
* n|shift+n
下一个关键字|上一个关键字\
* v
visual模式，需要选择文本的时候使用\
* y
复制选择的文本，退出vim时会消失\
* p
粘贴\
* gg
光标移动到第一行\
* G
光标移动到最后一行\

## tar 打包压缩
tar [options] tarFile path
* c
打包文件\
* x
解包文件\
* z|j
使用Gzip或Bzip2压缩或解压\
* v
显示压缩、解压过程\
* f
指定文件\

## 运行程序
* ./filenmae
运行\
* ctrl+c
退出\
* nohup cmd &
默认会生成nohup文件并将cmd输出写入该文件\
> filename\
重定向输出文件\
```
nohup cmd > other.log &
```
2>&1\
输出输入分为3种，2标准错误输出，1标准输出，0标准输入\
2>&1 表示把2定为1，把1再定为输入写入输入文件中\
简单来说就是把所有信息都存入log文件中\
* kill
退出程序\

## systemctl 服务方式启动
centOS的服务管理程序\
* systemctl enable serviceName
设置开启启动\
* systemctl disable serviceName
取消开机启动\
* systemctl start serviceName
启动服务\
* systemctl stop serviceName
关闭服务\

## 系统相关
* sata
显示文件的详细信息，比ls更详细\
* who
显示在线登录用户\
* whoami
显示当前操作用户\
* hostname
显示当前主机名\
docker中显示的就是当前的容器id\
* uname
显示系统信息\
* top
动态显示资源消耗最多的进程\
* ps
显示当前进程状态的快照\
* du
显示当前目录大小\
-h 显示单位，如k\
* df
显示当前磁盘大小\
* ifconfig
显示网络情况\
* ping
测试网络连通情况\
* netstat
显示网络状态信息\
* man cmd
查看命令文档
```
man grep
```
* clear
清屏\

## firewall 防火墙
防火墙服务\
firewall -cmd --cmdName\
执行防火墙相关命令\
* --state
状态\
* --version
版本\
* --reload
更新防火墙规则\
* --panic-on|--panic-off
打开或关闭关闭连接状态\
* --zone=[statusName]
作用域，比如开放的端口就是public\
* --list-ports
显示端口，一般和--zone连用，zone用以过滤\
* --add-port|--remove-port
添加或移除端口，一般和--zone连用，表示作用域\
* --permanent
永久生效，一般和--add-port等连用

## 关机和重启
* shutdown
关机\
-r 重启\
-h 不重启\
* rebot
重启\
* halt
关机\

## 用户管理
* useradd 
创建用户\
* passward userName
设置用户密码\
* adduser
创建用户并自动创建主目录等所需操作的useradd的便利脚本\
* userdel
删除用户\
* groupadd
创建组\
* groupdel
删除组\
* su userName
切换用户\
* sudo cmd
暂时拥有root权力执行命令\



# 常见问题

# sh文件无法打开
在windows中编辑的sh文件，在linux中无法打开\
```
# 报错信息
# /bin/bash^M: bad interpreter: No such file or directory
```
原因是在windows编辑完后换行符自动为\r，而linux中为空\
解决方法\
```
# 将\r结尾的字符替换为空，即可在linux中运行
sed -i 's/\r$//' /opt/start.sh
```