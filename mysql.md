# innodb|myisam 区别
* 都是使用B+树结构，但myisam是非聚簇索引，主键只记录了数据的引用，innodb则是和主键记录在一起
* innodb支持外键，myisam不支持
* innodb支持事务，每条sql都是包装成事务执行，myisam不支持事务
* innodb必须要有主键，myisam可以没有
* innodb的辅助索引和主键索引是层级关系，即辅助索引只能查询主键，而myisam是平级关系，辅助索引可以直接查到数据
* innodb不保存具体行数，select count(*) from tableName 会扫描全表，myisam则是保存了一个变量存储，执行很快
* innodb5.7前没有全文索引，myisam一直有
* innodb有表锁行锁，myisam只有表锁



# 聚簇索引和非聚簇索引

## 聚簇索引
记录数据结构构成的主要索引，且数据记录在叶节点上和主键绑定\
指按B+树的索引，大部分情况就是id

## 非聚簇索引
又称为辅助索引，为了辅助查找聚簇索引而建立的索引\
非聚簇索引实际是新建了一个B+树或哈希等结构重新构建了一个索引，而索引的叶节点为主键值\
因此如果是相等性搜索，哈希索引更有利，如果是范围搜索B+树索引更有利



# 事务
事务是数据库逻辑单元，每次执行实际就是事务的执行

## 事务特性
* 原子性\
事务是数据库逻辑单元，事务内的要么都做，要么都不做
* 一致性\
事务执行结果必须让数据库从一个一致性状态转为另一个一致性状态
* 隔离性\
事务执行不能互相干扰
* 持久性\
事务一旦提交，改变就是永久性的

## 事务隔离级别
* 读未提交\
当前事务读取到其他事务未提交数据\
缺点：有可能其他事务回滚导致脏读
* 读已提交\
当前事务读取其他事务已提交数据\
缺点：解决了脏读问题，但有可能同一个事务中相同语句前后读取到不同数据
* 可重复读\
在同一事务中多次读取到的数据是一致的\
缺点：有可能读取不到最新数据
* 串行化\
事务串行化，阻塞其他事务执行，避免事务冲突\
缺点：事务无法并发，执行效率低



# 锁

## 全局锁
一般用于数据备份的情况下，让整个数据库处于只读状态\
优化：\
可通过开启可重复读隔离级别开始下一个事务，这样可以获取一个一致性视图用以导出

## 表级锁
* 表锁
```sql
lock tables table_name_1 read, table_name_2 write;
unlock tables table_name_1,table_name_2;
```
* 元数据锁\
不需要显式使用，在访问表时会自动加上，用以保证读写正确性\
对一个表进行增删改时添加读锁，对表结构更改时添加写锁\
读锁不互斥，可以多线程对同一表进行增删改

## 行锁
* 共享锁\
锁定资源可以读，不能写，一般在select执行时启动，数据读取完毕后释放
* 排他锁\
除当前线程外均不能操作，在insert、delete、update时会自动启用，会持续到事务结束
* 更新锁\
为防止死锁而设计，当执行更新时，会先加更新锁，数据还可以读单不能写，在执行更新时转为排他锁

## 间隙锁
避免读取到新插入的行（幻读），会在行与行直接添加间隙锁，避免插入

## 锁协议
```sql
update table_name set field = 1 where id in (2,3);
```
如果id没有索引，则使用表锁，然后查询数据\
如果id有索引，则会启用行锁，锁定2行数据\
事务结束后释放锁

## 死锁
有两个不同事务，A事务先锁定了a行，想获取b行锁，B事务锁定了b行，想获取a行锁，此时就发生了死锁\
死锁解决方案：\
按顺序获取锁\
死锁检测，设置innodb_lock_wait_timeout，发生死锁并超时后主动回滚事务让其他事务执行，通过设置innodb_deadlock_detect=on开始

# 分库分表

## 垂直分表
将热字段单独存放在另一张表中，减少单表字段量\
优点：\
解决系统层面的耦合，业务清晰\
类似微服务对不同的数据进行分级管理、维护\
高并发情况下可以提高一定的IO，单机硬件瓶颈\
缺点：\
部分表无法join，只能通过接口聚合解决\
分布式事务处理更复杂\
单表依然存在数据量过大的问题

## 水平分表
按分类id、user_id等进行分表，经过散列函数将一个表拆到多库多表中
优点：\
不存在单库数据量过大问题\
应用端改造较小\
缺点：\
跨分片的数据一致性，比如不查询user_id这时就需要同时在4库发起查询，再在内存中合并数据，分库反而成为拖累\
跨库join性能较差\
数据多次扩展维度，维护量极大



# 主从复制
主库提供读写，从库提供读\
优点：\
高可用，主库挂了可以切换至从库\
读写分离，降低主库压力\
备份数据

## 主从复制过程
主库开启bin log，记录所有增删改操作\
从库发起连接，连接到主库\
主库创建binlog dump thread，将bin log内容发送给从库\
从库将数据复制到自己的relay log中\
从库通过relay log重做日志文件并执行相关sql

### 数据复制方案
* 异步复制：\
主库将事件发送给从库，不关数据是否同步成果\
缺点：可能会造成数据丢失
* 同步复制：\
主库将事件发送给从库，等待所有从库返回数据复制成功消息\
缺点：性能差
* 半同步复制：\
主库将事件发送给从库后，等待任何一个从库返回数据复制成功消息

## 其他特殊关系
* 主主：两边都是主库，数据双向同步
* 主备：备库只更新数据，不提供数据读取

## 主从一致

### 长连接
主从维持长连接

### binlog格式
* statement\
记录sql原文\
缺点：\
当主从数据库索引不一致时，sql执行不一致
```sql
delete from table_name where field_1 > 1 and field_2 < 1 limit 1;   //当主库索引field_1从库索引field_2时数据不一致
```
* row\
不记录原文，而是记录table_map和delete_rows两个event，前者记录要修改的表，后者记录删除的行为\
缺点：\
当操作数据量大时，日志占用大量空间
* mixed\
statement和row的混合\
当不会出现数据不一致时使用statement，否则使用row

## 主从延迟
主要是指从主数据库执行事务并记录binlog到从数据库同步完事务之间的时间差

### 可能原因
* 从数据库硬件性能差
* 从库读取压力大导致性能下降
* 执行需要较长时间的大事务，对语句进行分批处理
* 网络延迟
* 从库过多
* mysql版本低，只能使用单线程复制



# 分布式事务
根据Seata中的事务类型进行阐述

## XA事务
强一致性分阶段事务模式，牺牲可用性换取安全性

### 两阶段XA
事务协调者进行prepare请求，事务参与者提交事务回复yes或no\
若所有都回复yes则发送commit请求，否则发送rollback请求\
缺点：\
prepare阶段就锁定了资源，如果其他事务也需要更改相同数据就只能等待事务完成，高并发情况下性能下降\
prepare阶段后事务协调者宕机，会导致无限期等待事务协调者的commit或rollback命令\
prepare阶段成功，但事务协调者发送给某参与者的命令发送失败，会出现数据不一致

### 三阶段XA
将prepare分为cancommit及precommit阶段，引入超时机制及状态确认，若未收到协调者的指令，超时后参与者仍会执行commit\
缺点：\
在协调者发送rollback指令给某节点时失败，这个节点在超时后仍会执行commit导致数据不一致

## AT事务
最终一致性分阶段事务

### AT事务过程
在参与者中注册事务\
参与者记录数据快照\
参与者执行业务并提交，释放数据锁\
参与者向协调者报告事务状态\
协调者统合参与者事务状态发送commit或rollback指令\
参与者接收到commit指令就删除数据快照，接受rollback指令就根据数据快照恢复数据

优点：\
较XA有比较好的性能提升\
缺点：\
最终一致\
多线程并发可能出现脏写，具体表现为事务一修改为60，事务二修改为80，事务一回滚为100，事务二修改丢失，解决方案是在事务结束前引入全局锁\
快照仍会影响性能，但较之XA好很多

## TCC事务
通过人工编码实现类似AT的事务

### TCC事务过程
以账户金额为例\
try阶段获取账户金额是否充足，充足则将使用部分移至冻结金额，账户金额扣除冻结金额\
confirm时直接扣减冻结金额，cancel时将冻结金额重新加到账户金额

优点：\
相比AT事务没有快照没有全局锁，性能好\
不依赖数据库事务，而是依赖补偿逻辑\
缺点：\
需要人为编写相关接口逻辑\
最终一致\
需要考虑confirm及cancel情况，做好幂等处理（操作一次或多次请求结果一致，不会因多次操作产生副作用）

### TCC事务的相关问题
* 空回滚\
某分支try未执行直到事务超时，此时进行事务回滚，此分支其实不用执行cancel操作\
解决方案：cancel时判断是否已经执行try操作，未执行try操作的不执行cancel操作
* 业务悬挂\
某分支已经进行空回滚，但此时try操作执行，但事务总体已经结束，导致此分支事务一直处于中间状态\
解决方案：执行try时判断是否已经执行cancel操作。已执行cancel则不执行try操作

## SAGA事务
当无法满足实现TCC事务的3个方法时，且需要依次执行事务时

### SAGA事务过程
依次执行参与者的正向操作\
所有参与者执行成果则什么也不处理直接提交，否则执行前面各参与者的逆向回滚操作

优点：\
无快照无锁性能好\
相比TCC事务实现更简单\
缺点：\
软状态持续时间不确定，时效性差\
会出现高并发脏写



# 索引优化（非聚簇索引优化）

## 索引数据结构
* B+树
* R树\
一般用于空间上的查找
* HASH索引
* 倒序索引\
用于全文索引，一般使用哈希加链表或树形词典的数据结构\
根据分词算法将记录进行分词，然后用数据结构保存起来

## 索引种类类型
* 主键索引
* 普通索引
* 联合索引
* 唯一索引
* 全文索引
* 前缀索引

## 前缀索引
```sql
table table_name add index index_name(field_name(index_length));
```
使用某字段字符串的前几个字符建立索引\
优点：\
减少索引大小\
提高查询效率\
缺点：\
orderBy无法使用前缀索引\
前缀索引无法进行索引覆盖

## 全文索引
PS：本篇特指mysql的全文索引，其他全文索引算法请见 C#/Learn Journey - algorithm \
主要通过倒排索引实现，倒排索引主要存储了单词与单词在一个或多个文档间的映射，mysql采用full inverted index，能通过单词找到文档，并从文档找到对应的位置

### 建立全文索引
只能建立在char、varchar、text上
```sql
ALTER TABLE article ADD FULLTEXT INDEX fulltext_article(title,content);
```

### 全文索引的搜索条件
* 最小搜索长度|最大搜索长度\
全文索引只能搜索在这个区间范围内的\
默认情况下innodb是3-84，myisam是4-84，可以进行配置
* 全文索引必须和定义时所用的字段一致，fulltext_article(title,content)，搜索时必须是match(title,content)，如果要使用全文索引分别搜索title和content，就需要在两个字段分别建立全文索引

```sql
select * from article match(title,content) against('search');
```

### 内置中文全文索引
mysql内置索引器可以通过设置ngram_token_size配置匹配值，范围为2-10，实际这个匹配值就是n元分词法，如果设为2就不能进行单个字的搜索\
ps:其他分词法，比较主流的是mmseg分词

### 自定义中文全文索引
自定义中文全文索引一般使用二元分词法或其他方法方法进行分词操作\
再将分词操作的词转换为汉字区位码或其他形式以转换为符合mysql英文全文索引的字符串

### 全文模式的相关性
相关性的高低代表搜索结果排序的高低\
相关性和词汇辨识度成反比，词汇辨识度是词汇在索引中的出现次数\
相关性和词汇出现次数成正比，即单条结果中词汇的出现次数越多，相关性越高

### 全文索引模式

#### NATURAL LANGUAGE MODE
自然索引的模式\
mysql 默认的全文索引模式\
以相等的形式进行搜索
```sql
//以下三条是等价的
select * from article match(content) against("武汉 雨伞");
select * from article match(content) against("武汉 雨伞" in NATURE LANGUAGE MODE);
select * from article match(content) against("武汉 雨伞" in BOOLEAN MODE);
```

#### BOOLEAN MODE
布尔模式\
布尔模式下可以对搜索条件进行修改\
* +:必须出现
* -:不能出现
* 无或>:增加相关性
```sql
//同时有武汉雨伞的相关性最高会排在最前
//只出现武汉或雨伞的会排在第二位
select * from article match(content) against("武汉 雨伞" in BOOLEAN MODE);
```
* <:减少或增加相关性
```sql
//只出现武汉的相关性最高，会排在最前
//同时出现武汉和雨伞的相关性会降低，排在第二位
//只出现雨伞的相关性最低，排在第三位
select * from article match(content) against("武汉 <雨伞" in BOOLEAN MODE);
```
@distance_int:必须满足一定的编辑距离
* ~:负相关性
```sql
//只出现武汉的相关性最高，会排在最前
//同时出现武汉和雨伞的相关性会降低，排在第二位
//只出现雨伞的不会会匹配到
select * from article match(content) against("武汉 ~雨伞" in BOOLEAN MODE);
```
* *:通配符\
mysql针对英文的全文匹配默认的单词匹配是等值匹配\
*可以提供通配性查询
```sql
//可以匹配agent
select * from article match(content) against("age*" in BOOLEAM MODE);
//只能匹配age
select * from article match(content) against("age" in BOOLEAM MODE);
```
* "":短语\
双引号内内容以短语形式进行检索
```sql
//只能匹配武汉雨伞
select * from article match(content) against('"武汉雨伞"' in BOOLEAN MODE);
```

#### QUERY EXPANSION
拓展搜索模式\
把查询结果当查询条件再进行一次查询
```sql
select * from article match(content) against("武汉" in QUERY EXPANSION);

//若搜索结果为"武汉市是好地方" 则等价于以下
select * from article match(content) against("武汉市是好地方");
```


## 覆盖索引
查询字段完全在索引范围中，避免通过索引查询到主键id，再通过主键id查询数据的回表操作

## 主键自增
主键连续使得更新时不需要移动数据，插入效率高\
非自增索引会导致B+树页分裂，出现大量内存碎片，影响查询速度

## 索引失效
* 使用左模糊或左右模糊匹配。%like|%like%
* 对索引列进行计算、函数或类型转换
* 不满足最左原则\
abc索引只查询ac，只能利用a\
abc索引查询abc，但b使用<、>、between、like，则只能利用ab
* 在where语句中，如果使用or，or前或后有一个不是索引，就会导致索引失效\
or使用索引的本质就是使用了union语句，如果a是索引b不是，那么b需要用全表扫描，既然使用全表扫描，单次全表扫描ab字段显然优于扫描a索引加全表扫描b字段\
同理复合索引abc，a = and (b or c)情况下只命中a索引\
对同一个索引字段使用or在mysql5.7以前的版本会使索引失效

## 执行计划|explain
* possible_keys 可能用到的索引
* key 实际用到的索引
* key_len 索引长度
* rows 扫描数据行数
* type 扫描数据的类型
const 结果只有1条且用主键或唯一索引扫描\
eq_ref 唯一索引扫描\
ref 非唯一索引扫描\
range 索引范围扫描\
index 全索引扫描，相当于没有查询条件的索引扫描
```sql
select id from table_nmae
```
all 全表扫描
```sql
select * from table_nmae
```
* extra 执行情况的描述说明\
use index 索引覆盖\
use temporary 产生临时表保存中间结果，一般是由于order by或group by导致，比如order by和group by使用不同列，或者使用order by或group by时对第一个表以外的表使用连接查询

### desc
和explain作用基本一致

## 适合建立索引的字段
字段唯一\
经常使用where查询\
用于groupby或orderby字段

## 不适合建立索引的字段
字段有大量重复数据\
表数据太少\
经常更新的字段

## 添加索引但使用全表扫描
mysql优化器会评估走索引的效率和全表扫描的效率，当全表扫描效率大于走索引的效率时会直接走全表扫描\
一般是由于需要扫描的数据量在总表数据量中占比较大

## 唯一索引与普通索引
唯一索引的优点：\
查询比普通索引稍快，在查询到数据后立刻返回，普通索引会查询下一条直到值不一样\
数据唯一\
唯一索引的缺点：\
每次更新或插入都需要验证唯一性，无法使用change buffer，而普通索引的字段在更新操作时会先将更新操作存入change buffer中
> change buffer
> 非唯一的字段在更新时会先将更新操作存入change buffer中，当表数据需要读取时再执行change buffer里的操作，避免高频率从磁盘读取数据，减少了磁盘io，因此存入数据后到读取数据的时间间隔越长，越能从这个机制中获利\
> 缺点：当数据存入后立即需要读取的情况，change buffer反而是额外的开销
> 


## in|not in
in会走索引，not in不走索引



# 分页优化
需要分页优化的原因：\
使用limit m,n实际是查出m+n条数据后抛弃了m条，则其实仍然有大量的io操作，可以理解为select是早于limit执行的，所以会通过聚簇索引查询出大量叶节点的其他数据

## id 限定优化
如果id是连续不断的，那么可以使用id进行查询\
直接减少了数据查询量
```sql
//实际是从1000000开始查
select * from a where id between 1000000 and 1000100 limit 100;
```

## 使用子查询优化
子查询优化主要是优化了无用的叶节点数据被查询出来的过程
```sql
//相当于省去了前1000000条数据通过id查询*的过程
select * from a where k = 100 and id >= (select id from a where k = 100 limit 1000000,10) limit 10;
```

## 基于索引在排序
利用了索引查询中的优化算法，通过索引找相关数据地址可以避免全表扫描，一般用于主键索引或是唯一索引
```sql
select * from a order by id desc limit 1000000,10;
```



# 常见数据存储

## 手机号
* int 不可以，因为int只能取到0~2^32-1，精度只有10位，手机号有11位
* bigint 可以，且只用8字节，相当于long，优点是空间小
* char[11]  可以，看具体编码，常用的utf-8及gbk中数字都是1字节，因此要11字节，优点是定长查询快



# 整形长度的作用
在整数类型下，长度只有在和zerofill连用时才有作用，其他时候没有作用\
zerofill会在数字显式长度不足时在前面补0



# 常用sql

## exist 对比 in
in先查询in的内容，exist先查询exist外的内容，相当于B驱动A和A驱动B，一般B较小时使用in，A较小时使用exist

## leftjoin、rightjoin、innerjoin
* leftjoin\
A leftjoin B\
会查出A中存在B中不存在的数据或AB中都存在的数据
A leftjoin B where B.id = null\
会查出A中存在且B中不存在的数据
* rightjoin\
A rightjoin B\
会查出B中存在A中不存在的数据或AB中都存在的数据\
A rightjoin B where A.id = null\
会查出B中存在且A中不存在的数据
* innerjoin\
会查出AB中都存在的数据

## union、union all、or
* union、or对比
```sql
select name,population,area from World where area>=3000000
union
select name,population,area from World where population>=25000000;
```
```sql
select name,population,area from World where area>=3000000 or population>=25000000;
```
or命中索引的本质是使用union命中了索引\
在两字段都建立索引的情况下，or和union效率一致\
在单字段建立索引或没有字段建立索引的情况下，or效率高于union，因为or只需要执行一次全表搜索就能得到结果
* union、union all 对比\
union会在数据合并后去重，union all跳过去重步骤

## and or 优先级
and高于or，在没有括号的情况下，and先执行



# 常见问题

## mysql8身份验证插件导致的登陆失败
mysql8默认的身份认证插件是caching_sha2_password，之前的是mysql_native_password\
* 解决方法：
1. 修改my.conf中mysqld中
```conf
default_authentication_plugin=mysql_native_password
```
2. 重启mysql
3. 登陆mysql修改用户登陆验证方式
```sh
use mysql;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
FLUSH PRIVILEGES;   //刷新权限
```
4. 扩大权限范围
```sh
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
FLUSH PRIVILEGES;   //刷新权限
```



# 外置应用索引
相当于用其他应用对mysql建立额外的索引\
优势：
* 实现mysql无法实现的复杂索引需求，比如对分库分表后的mysql库进行非分库依据字段的全库检索
* 依赖高速内存，避免较慢的磁盘io
* 优化存储模式

## es
java内核\
使用lucene做索引与搜索\
分布式搜索引擎架构

### lucene
底层使用分段存储的模式，在读写上完全避免锁，提高读写性能\
es基于lucene开发，隐藏lucene的复杂性，并提供了简单的restful接口

#### lucene的索引写入流程
* 从数据库读取并进行分词
* 通过分词创建索引
* 存储索引，有hash和FST两种模式，hash更快但占用空间更大，一般使用FST存储

#### lucene的索引搜索流程
* 分析查询语句，进行分词处理
* 将分词处理为lucene可识别条件语句
* 通过search模块开始搜索流程
* 通过store从以保存的索引中获取相关内容
* 执行相关性打分并排序

#### lucene的搜索方式
* 按搜索语句的分词，从索引库中查找相同的词汇，由此得出对于词汇所关联的数据的id数组
* 按and、or、not分别进行交集、并集、差集操作
* 得出对应的数据id结果集

#### 增量更新的原理
为了避免每次数据修改都将索引文件重新更新，推出的解决方法\
段：引入段的概念，将索引文件拆分为多个子文件，每个子文件既是段，段不可修改
* 新增\
新增一个段来存储新内容
* 删除\
新增一个删除记录保存被删除索引的id，不更改段文件\
搜索时原记录仍会被查询，但会被过滤\
在合并段时，才会执行删除，但实际是删除了原段文件，生成了新的
* 更新\
新建一个删除记录，并新增段来存储新内容

#### 写入与准实时引擎
为了避免每次新建操作都产生一个段文件，导致过多的文件使得查询效率下降，引入了类似于redis固化AOF的策略\
当新数据被增加时会写入内存缓冲区，内存缓冲区只能写不能读，因此这部分新增索引其实没有生效，也无法被查询到，因此是准实时\
为了避免AOF策略导致的索引因不可控情况丢失，es增加了事务日志保存记录

#### 段合并策略
* 超过一定大小的段无法执行合并，因为大段合并需要很多性能
* 大小在一个相同区间的段才会执行合并
* 段合并主要集中在中、小段文件上

### es核心概念
* 集群\
由多个es节点构成
* 节点\
单个es服务单元，名称不能重复
* 分片\
当单个索引数据量过大时，可以将数据进行水平拆分，拆分出的数据库被称为分片\
分片数量在索引创建时指定，且不能更改\
提高并行能力\
相当于mysql分库，数据保存方式也和分库分表一致，对数据进行散列函数计算，分到不同的分片中\
每个分片节点都可以接收查询请求，此时该节点被称为协调节点，协调节点会通过散列函数计算出需要查询的分片并请求获取搜索结果，返回给客户端
* 备份分片\
类似于mysql的主从一致\
提高系统可用性
* 索引\
由一个或多个分片构成\
名称唯一\
类似于Mysql的数据库
* Type\
类型是索引逻辑上的分区\
类似于Mysql的表
* Field\
字段\
相当于Mysql的字段
* Mapping\
映射，定义索引的文件，主要用于自定义分词方法，其他情况下数据会自动识别\
创建后不可更改
* Analyzer\
分词方法
* 集群状态\
Green  所有节点都可正常使用\
Yellow  所有主节点可以正常使用，但有一个或多个备份节点无法正常使用\
Red  一个或多个主节点无法正常使用，且其备份节点也无法正常使用，搜索仍可进行，但只能查出正常部分的数据

### es客户端操作
一般可以通过三种方式操作es
* 使用elasticsearch-head插件
* 使用es提供的Restful接口
* 使用es提供的api

### es Restful接口
* {put} /{index_name}\
    创建索引
* {delete} /{index_name}\
    删除索引
* {put} /{index_name}/_mapping\
    创建映射
    ```json
    //请求体
    "properties": {
        "id": {     
            "type": "long",
            "store": true,
            "index": false
        },
        "title": {
            "type": "text",
            "store": true,
            "index": true,
            "analyzer": "standard"
        },
        "content": {
            "type": "text",
            "store": true,
            "index": true,
            "analyzer": "standard"
        }
    }
    ```
* {post} /{index_name}/{id}\
    创建或修改文档，即添加或修改数据
    ```json
    //请求体
    {
        "id": 1,
        "title": "ElasticSearch是一个基于Lucene的搜索服务器",
        "content": "它提供了一个分布式多用户能力的全文搜索引擎。"
    }
    ```
* {delete} /{index_name}/{id}\
    删除文档
* {get} /{index_name}/id\
    查询文档



### 脑裂
es通过心跳判断主节点存活情况，当心跳故障但主节点存活的情况，由于会推选出新的主节点，但是多个主节点共存，有可能发生数据丢失

### es重启
将持久化段加载到内存中\
将事务日志中未执行完毕的加载到内存中

### es|mysql的区别
* es使用document格式储存，无需标明字段，mysql需要
* es是分布式架构，mysql不是
* es是非实时的，mysql是实时的
* 数据存储性能，mysql由于索引原因，在数据库量过大时性能会衰退，es只需要足量内存
* 查询性能，es默认每个输入字段都是有索引的，因此各字段组合查询性能很好，复杂的关联查询或精确检索或覆盖索引的情况下mysql性能好

es在部分情况下优于mysql的原因
* 全文搜索，大幅度减少IO
* 通过分片降低检索规模，提升并行率

### es|sphinx 对比
* es导入数据生成索引的速度慢于sphinx
* es支持增量更新，sphinx不支持，而是提供了一个辅助表的解决模式，但会增加复杂度
* es可视化辅助工具Kibana(可视化通过数据生成索引)、Beats(日志收集工具)、Logstash(日志处理工具)，sphinx可视化辅助工具sphinx tools只能监控性能
* 算法方面没有太大差异
* es在查询方面支持QueryDSL(通过一个特定格式的json请求体进行查询的方式)
* es在排序方面的Function Score Query也比sphinx的更方便实现
* es更简单的实现分布式集群化，sphinx则更为复杂
* es资源占用多于sphinx，因为java在这部分劣于c++的关系

## sphinx
c++内核\
全文搜索引擎

### 使用场景
* 快速高效的核心全文索引
* 并行产生结果集\
在单次数据扫描中可以并行执行多个搜索返回多个结果集
* 向上、向外扩展\
向上扩展：增加cpu/内核，扩展磁盘\
向外扩展：分布式sphinx
* 聚合分片数据\
将多个mysql库的数据进行聚合，建立外置索引

### 工作流程
从mysql读取数据\
依据数据建立索引\
搜索数据返回结果\
根据结果从mysql进行数据读取

### 配置
* 数据源配置
```conf
source source_name      #数据源名称
{
    type                    = mysql     #数据源类型

    sql_host                = 127.0.0.1
    sql_user                = root
    sql_pass                = root
    sql_db                  = test
    sql_port                = 3306    #默认为3306

    sql_query_pre           = SET NAMES utf8    #查询获取数据源时的编码
    sql_query       　　　　 = SELECT id, name, add_time FROM tbl_test    #查询获取数据源时的语句

    sql_attr_timestamp      = add_time    #sql_attr_* 索引属性，返回的搜索结果

    sql_query_info_pre      = SET NAMES utf8    #根据结果查询数据库结果时的编码
    sql_query_info          = SELECT * FROM tbl_test WHERE id=$id   #根据结果查询数据库结果时的语句
}
```
* 索引配置
```conf
index index_name    #索引名称
{
    source                    = test    #数据源名称
    path                      = /usr/local/coreseek/var/data/test   #索引文件的基本名，生成索引文件会是test1.spa...
    docinfo                   = extern  #索引文档属性值的存储格式
    charset_dictpath          = /usr/local/mmseg3/etc/  #中文分词时的词典目录
    charset_type              = zh_cn.utf-8    #数据编码类型
    ngram_len                 = 1   #分词长度 n元
    ngram_chars               = U+3000..U+2FA1F    #进行n元分词模式时的有效字符集
}
```

### Coreseek
在sphinx基础上开发，支持中文mmseg算法