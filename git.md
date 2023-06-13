# 常用命令
* init
初始化项目仓库\
* clone
对远程仓库项目进行克隆\
* pull
拉取远程分支并合并\
* fetch
拉取远程分支但不合并\
pull fetch 的区别:\
fetch会将远程分支的信息拉取过来，不改变本地分支的指针，而pull实际是将远程分支合并到本地分支，会改变本地分支的指针，一般使用fetch+merge是比pull更安全的做法\
* add
添加指定目录到暂存区\
* commit
将暂存内容添加到本地仓库\
* reset
回滚代码到某一版本\
--hard commit_id 指定commit_id版本\
--hard origin/branch 回滚到和指定远程分支一样\
HEAD^ 回滚到上一版本\
fileName  回滚某文件到某版本\
* relog
回滚到错误commitid可以使用relog恢复之前的commitid的hash\
然后荣国reset --hard 回滚到正确commit\
* branch
查看所有分支信息\
branch newBranchName 新建分支\
-d branchName 删除分支\
* status
分支状态\
* checkout
切换分支\
-b newBranchName 创建新分支并切换\
. 回滚未存到暂存区的修改\
-- fileName 回滚某文件\
* push
推送分支\
--force 本地于远程有差异，强制推送\
push <远程主机名> <本地分支名> : <远程分支名>  推送分支\
* merge
merge branchName 将分支合并到当前分支\
-no-ff 即使可以进行快速合并，也产生合并标志，方便代码控制\
* rebase
改变基底。\
比如b是从a拉出的分支，后续a进行了a2的提交，b进行了c、d的提交，此时切换到b，rebase a，则实际是回退到a刚切出b的状态进行a2的提交，再进行cd的提交，此时a、a2、c、d处于一条线上\
撤销某次提交，可以通过revase -i commitId，并把commit文件中的pick改为drop\
* diff
查看冲突内容\
* revert
撤销某次提交并针对此次操作新建一个commit\
-m 对merge节点撤销需要加\
* log
查看提交历史\
--reverse 逆向显示\
--author 只显示某个用于的提交记录\
* remote
远程操作\
add origin uri  添加远程版本库\
rm name 删除远程版本库\
rename oldName newName  修改远程版本库名称\
* config
显示当前git配置信息\
* rm
fileName 删除文件\
--cached fileName 删除暂存区文件\
* tag
-a tagName -m commitName  为某次提交标记别名\