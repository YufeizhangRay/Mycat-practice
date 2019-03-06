# mycat-practice  
  
##  mycat学习实践  
  
- [数据库性能瓶颈](#数据库性能瓶颈)  
- [大数据量数据库性能解决方案](#大数据量数据库性能解决方案)  
  - [读写分离](#读写分离)  
  - [分库分表](#分库分表)  
- [分布式数据存储中间件的核心流程](#分布式数据存储中间件的核心流程)  
- [什么是Mycat](#什么是mycat)  
  - [名词解析](#名词解析)  
- [Mysql基于binlog的主从复制原理](#mysql基于binlog的主从复制原理)  
  - [实现操作](#实现操作)  
  - [主从复制的延迟](#主从复制的延迟)  
- [Mycat目录结构](#mycat目录结构)  
- [Mycat配置文件详解](#mycat配置文件详解)  
- [表拆分规则](#表拆分规则)  
  - [分片取舍](#分片取舍)  
  - [全局序列](#全局序列)  
- [Mycat注解](#mycat注解)    
- [Mycat命令行监控工具](#mycat命令行监控工具)  
- [Mycat弱XA事务机制](#mycat弱XA事务机制)
- [Mycat节点扩容](#mycat节点扩容)  
- [Mycat高可用](#mycat高可用)  
  - [Mycat高可用方案Haproxy安装和配置](#mycat高可用方案haproxy安装和配置)  
  
### 数据库性能瓶颈  
  
![](https://github.com/YufeizhangRay/image/blob/master/Mycat/%E5%8D%95%E6%95%B0%E6%8D%AE%E5%BA%93.jpeg)  
  
>1.数据库连接数不足(Mysql默认100个)   
2.表数据量大【空间存储的问题】  
>>索引  
命中不了，会进行全表的扫描。  
命中索引，硬盘级索引，它是存储在硬盘里面，要执行IO操作。当B-tree越来越深，则查找耗时增加。  
  
>3.硬件资源(QPS/TPS)  
  
### 大数据量数据库性能解决方案  
  
>1.sql优化  
2.缓存  
3.建好索引  
4.读写分离  
5.分库分表  
  
#### 读写分离  
区别读、写多数据源方式进行数据的存储和加载。 
数据的存储(增删改)一般指定写数据源，数据的读取查询指定读数据源(读写分离会基于主从复制)。
![](https://github.com/YufeizhangRay/image/blob/master/Mycat/%E8%AF%BB%E5%86%99%E5%88%86%E7%A6%BB.jpeg)  
   
主从形式  
![](https://github.com/YufeizhangRay/image/blob/master/Mycat/%E4%B8%BB%E4%BB%8E%E6%96%B9%E5%BC%8F.jpeg)  
  
解决问题  
>1.数据库连接   
2.硬件资源限制（QPS\TPS）  
  
#### 分库分表 
对数据的库表进行拆分，用分片的方式对数据进行管理。  
  
垂直拆分  
![](https://github.com/YufeizhangRay/image/blob/master/Mycat/%E5%9E%82%E7%9B%B4%E6%8B%86%E5%88%86.jpeg)  
  
解决问题  
>1.数据库连接   
2.硬件资源限制（QPS\TPS）  
  
带来问题  
>1.分布式事务问题  
2.跨库查询  

水平拆分  
![](https://github.com/YufeizhangRay/image/blob/master/Mycat/%E6%B0%B4%E5%B9%B3%E6%8B%86%E5%88%86.jpeg)  
  
解决问题  
>1.表数据量的存储空间也解决了  
2.数据库连接  
2.硬件资源限制（QPS\TPS）  
  
带来问题  
>1.跨库查询  
2.拆分规则  
  
### 分布式数据存储中间件的核心流程  
  
![](https://github.com/YufeizhangRay/image/blob/master/Mycat/%E6%A0%B8%E5%BF%83%E6%B5%81%E7%A8%8B.jpeg)  
  
### 什么是Mycat  
  
Mycat 是开源的分布式数据库中间件，基于阿里的cobar的开源框架之上。它处于数据库服务与应用服务之间。它是进行数据处理与整合的中间服务。  
  
通俗点讲，应用层可以将它看作是一个数据库的代理(或者直接看成加强版数据库)。  
![](https://github.com/YufeizhangRay/image/blob/master/Mycat/Mycat%E4%BD%8D%E7%BD%AE.jpeg)  
  
#### 名词解析  
![](https://github.com/YufeizhangRay/image/blob/master/Mycat/Mycat%E8%AE%BE%E8%AE%A1.jpeg)  
  
```
逻辑库  db_user db_store
逻辑表 	  
  分片表   用户表        用来分片的表。
  全局表   数据字典      修改不频繁且与分片表相关。
  ER表    用户地址表     用户和用户地址具有相关关系，我们不希望它们分开存储。
非分片表   门店表 店员表  没有进行分片且与分片表没有直接关系的表。

分片规则 userID%2 

节点    三个节点 两个分片db_user节点 主从db_store算一个节点
节点主机 写、读节点主机  两个粉色区域各表示一个节点主机 
```
  
### Mysql基于binlog的主从复制原理   
  
![](https://github.com/YufeizhangRay/image/blob/master/Mycat/%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6%E5%8E%9F%E7%90%86.jpeg)  
  
>1.master将操作记录到二进制日志(binary log)中(这些记录叫做二进制日志事件，binary log events)。  
>2.Slave通过I/O Thread异步将master的binary log events拷贝到它的中继日志(relay log)。  
>3.Slave执行relay日志中的事件，匹配自己的配置将需要执行的数据，在slave服务上执行一遍从而达到复制数据的目的。  
  
#### 实现操作  

Master操作: 
```
1.接入mysql并创建主从复制的用户  
create user m2ssync identified by 'Zyf123!@#';   
2.给新建的用户赋权  
GRANT REPLICATION SLAVE ON *.* TO 'm2ssync'@'%' IDENTIFIED BY 'Zyf123!@#';   
3.指定服务ID，开启binlog日志记录，在my.cnf 中加入  
server-id=137  
log-bin=dbstore_binlog  
binlog-do-db=db_store  
4.通过SHOW MASTER STATUS;查看Master db状态.  
```
  
Slave操作:   
```
1.指定服务器ID，指定同步的binlog存储位置，在 my.cnf中加入  
server-id=101  
relay-log=slave-relay-bin  
relay-log-index=slave-relay-bin.index  
read_only=1  
replicate_do_db=db_store  
2.接入slave的mysql服务，并配置  
change master to master_host='192.168.8.137',   
master_port=3306,  
master_user='m2ssync',  
master_p assword='Zyf123!@#',  
master_log_file='db_stoere_bi nlog',  
master_log_pos=0;  
>3.start slave;  
>4.show slave status\G;查看slave服务器状态  
```
  
#### 主从复制的延迟  
>1.当master  tps高于slave的sql线程所能承受的范围  
2.网络原因  
3.磁盘读写耗时  
  
判断延迟  
>1.show slave status \G;
```
sends_behind_master  0   越小越好，0表示没有延迟。
```
>2.mk-heartbeat  
```
timestamp进行实践搓的判断  
```
  
延迟问题解决  
>1.配置更高的硬件资源  
>2.把I/Othread改变成多线程的方式  
>3.应用程序自己去判断（mycat有这个方案） 
  
### Mycat目录结构  
  
>bin 程序目录，存放了 window 版本和 linux 版本可执行文件./mycat {start|restart|stop|status...} conf 目录下存放配置文件，  
>>server.xml 是 Mycat 服务器参数调整和用户授权的配置文件  
schema.xml 是逻辑库定义和表  
rule.xml 是分片规则的配置文件，分片规则的具体一些参数信息单独存放为文件，也在这个目录下  
log4j2.xml配置logs目录日志输出规则  
wrapper.conf JVM相关参数调整  
  
>lib 目录下主要存放 mycat 依赖的一些 jar 文件  
logs目录日志存放日志文件  
  
### Mycat配置文件详解
  
```
schema.xml 逻辑库表

schema 逻辑库

sqlMaxLimit 	返回的数据量

table 		分片表
rule 		分片规则
primaryKey 	主键
autoIncrement 	全局ID自增长
needAddLimit	返回的数据量 可以覆盖sqlMaxLimit
childTable	ER表 

dataHost 节点主机

balance：负载/读写分离均衡类型。
取值为
balance="0"，不开启读写分离机制，所有读操作都发送到当前可用的 writeHost 上。
balance="1"，全部的 readHost 与 stand by writeHost 参与 select 语句的负载均衡，简单的说，当双主双从模式(M1->S1，M2->S2，并且 M1 与 M2 互为主备)，正常情况下，M2,S1,S2 都参与 select 语句的负载均衡。
balance="2"，所有读操作都随机的在 writeHost、readhost 上分发。

switchType：主从切换策略
-1 表示不自动切换
1 默认值，自动切换
2 基于 MySQL 主从同步的状态决定是否切换
心跳语句设置为 show slave status
通过检测 show slave status 中的 "Seconds_Behind_Master","Slave_IO_Running","Slave_SQL_Running" 三个字段以及slaveThreshold 设置的值进行判断是否进行主从切换。

usingDecrypt 
可加入usingDecrypt属性来指定密码加密。1开启，0不开启
进入lib目录的Mycat-server-1.6-RELEASE.jar 执行
java -cp Mycat-server-1.6-RELEASE.jar io.mycat.util.DecryptUtil 	1:host:user:password
1:host:user:password 中 1 为 db 端加密标志，host 为 dataHost 的 host 名称

dataNode 节点


server.xml 启动参数

sequnceHandlerType：全局序列的方式
Processors 进程数
processorExecutor 进程中的线程数

firewall 黑白名单设置

user
benchmark  当前端连接达到设置的值，不再允许这个用户进行接入
privileges 表级的DML权限控制。4位2进制顺序为 insert update select delete 


rule.xml 拆分规则 下面单独叙述
```
  
### 表拆分规则  
  
```
连续分片  
优点:扩容无需迁移数据，范围条件查询资源消耗小  
缺点:数据热点问题，并发能力受限于分片节点
代表:
• 按日期(天)分片
• 自定义数字范围分片 
• 自然月分片
```
```
离散分片 
优点:数据分布均匀，并发能力强，不受限分片节点
缺点:移植性差，扩容难
代表:
• 枚举分片
• 数字取模分片
• 字符串hash分片
• 一致性哈希分片 
• 程序指定
```
```
综合类分片
兼并二者
代表:
• 范围求模分片
• 取模范围约束分片
```
```
自定义数字范围分片，提前规划好分片字段某个范围属于哪个分片
<function name="rang-long" class="io.mycat.route.function.AutoPartitionByLong">
  <property name="mapFile">autopartition-long.txt</property>
  <property name="defaultNode">0</property> 
</function>

defaultNode 超过范围后的默认节点。

此配置非常简单，即预先制定可能的id范围到某个分片 0-500M=0
500M-1000M=1
1000M-1500M=2
或
0-10000000=0
10000001-20000000=1
注意: 所有的节点配置都是从0开始，及0代表节点1
```
```
按日期(天)分片: 从开始日期算起，按照天数来分片
<function name=“sharding-by-date” class=“io.mycat.route.function.PartitionByDate"> 
  <property name=“dateFormat”>yyyy-MM-dd</property> <!—日期格式--> 
  <property name=“sBeginDate”>2014-01-01</property> <!—开始日期--> 
  <property name=“sPartionDay”>10</property> <!—每分片天数-->
</function>

按日期(自然月)分片: 从开始日期算起，按照自然月来分片
<function name=“sharding-by-month” class=“io.mycat.route.function.PartitionByMonth"> 
  <property name=“dateFormat”>yyyy-MM-dd</property> <!—日期格式--> 
  <property name=“sBeginDate”>2014-01-01</property> <!—开始日期-->
</function>

注意: 需要提前将分片规划好，建好，否则有可能日期超出实际配置分片数。
```
```
按单月小时分片:最小粒度是小时，可以一天最多24个分片，最少1个分片，一个月完后下月从头开始循环。
<function name="sharding-by-hour" class=“io.mycat.route.function.LatestMonthPartion"> 
  <property name=“splitOneDay”>24</property> <!-- 将一天的数据拆解成几个分片-->
</function>

注意事项:每个月月尾，需要手工清理数据
```
```
枚举分片:通过在配置文件中配置可能的枚举id，自己配置分片，本规则适用于特定的场景，比如有些业务需要按照省份或区县来做保存，而全国省份区县固定的。
<function name="hash-int" class=“io.mycat.route.function.PartitionByFileMap"> 
  <property name="mapFile">partition-hash-int.txt</property> 
  <property name="type">0</property>
  <property name="defaultNode">0</property>
</function>

partition-hash-int.txt 配置: 
10000=0
10010=1

mapFile标识配置文件名称
type默认值为0(0表示Integer，非零表示String) 
默认节点的作用:枚举分片时，如果碰到不识别的枚举值，就让它路由到默认节点
```
```
十进制求模分片:规则为对分片字段十进制取模运算，数据分布最均匀。
<function name="mod-long" class=“io.mycat.route.function.PartitionByMod"> <!-- how many data nodes -->
  <property name="count">3</property> 
</function>
```
```
应用指定分片:规则为对分片字段进行字符串截取，获取的字符串即指定分片。
<function name="sharding-by-substring“ class="io.mycat.route.function.PartitionDirectBySubString"> 
  <property name="startIndex">0</property><!-- zero-based -->
  <property name="size">2</property>
  <property name="partitionCount">8</property>
  <property name="defaultPartition">0</property> 
</function>

startIndex 开始截取的位置 
size 截取的长度 
partitionCount 分片数量 
defaultPartition 默认分片

例如 id=05-100000002
在此配置中代表根据 id 中从 startIndex=0，开始，截取 siz=2 位数字即 05，05 就是获取的分区，如果没传默认分配到 defaultPartition
```
```
截取数字hash分片
此规则是截取字符串中的int数值hash分片
<function name="sharding-by-stringhash" class=“io.mycat.route.function.PartitionByString">
  <property name=length>512</property><!-- zero-based --> 
  <property name="count">2</property>
  <property name="hashSlice">0:2</property>
</function> 

length代表字符串hash求模基数，count分区数，其中length*count=1024 
hashSlice hash预算位，即根据子字符串中int值 hash运算
0 代表 str.length(), -1 代表 str.length()-1，大于0只代表数字自身 
可以理解为substring(start，end)，start为0则只表示0 
例1:值“45abc”，hash预算位0:2 ，取其中45进行计算 
例2:值“aaaabbb2345”，hash预算位-4:0 ，取其中2345进行计算
```
```
一致性Hash分片: 此规则优点在于扩容时迁移数据量比较少，前提分片节点比较多，虚拟节点分配多些。虚拟节点少的缺点是会造成数据分布不够均匀如果实际分片数量比较少，迁移量会比较多。
<function name="murmur" class=“io.mycat.route.function.PartitionByMurmurHash">
  <property name=“seed”>0</property><!-- 创建hash对象的种子，默认0-->
  <property name="count">2</property><!-- 要分片的数据库节点数量，必须指定，否则没法分片--> 
  <property name="virtualBucketTimes">160</property>
</function>

注意: 一个实际的数据库节点被映射为这么多虚拟节点，默认是160倍，也就是虚拟节点数是物理节点数的160倍
```
![](https://github.com/YufeizhangRay/image/blob/master/Mycat/%E4%B8%80%E8%87%B4%E6%80%A7hash.jpeg)  
  
```
1.hash(str) { return  0 -> 2^32 },将整个0-2^32的hash值，作为一个hash环。
2.取node唯一标示计算出hash值，该hash结果即node在整个hash环中的位置。
3.将数据进行hash计算之后，顺时针找对应的node，改node即为该数据的服务node。
```
```
范围求模分片:先进行范围分片计算出分片组，组内再求模。
优点可以避免扩容时的数据迁移，又可以一定程度上避免范围分片的热点问题
分片组内使用求模可以保证组内数据比较均匀，分片组之间是范围分片可以兼顾范围查询。

最好事先规划好分片的数量，数据扩容时按分片组扩容，则原有分片组的数据不需要迁移。
由于分片组内数据比较均匀，所以分片组内可以避免热点数据问题。

<function name="rang-mod" class=“io.mycat.route.function.PartitionByRangeMod"> 
  <property name="mapFile">partition-range-mod.txt</property> 
  <property name="defaultNode">32</property>
</function>

partition-range-mod.txt 以下配置一个范围代表一个分片组，=号后面的数字代表该分片组所拥有的分片的数量。 
0-200M=5 //代表有5个分片节点
200M-400M=6
400M-600M=6
600M-800M=8
800M-1000M=7
```
```
取模范围约束分片:
对指定分片列进行取模后再由配置决定数据的节点分布。
<function name="sharding-by-pattern" class=“io.mycat.route.function.PartitionByPattern">
  <property name="patternValue">256</property>
  <property name="defaultNode">2</property>
  <property name="mapFile">partition-pattern.txt</property>
</function>

patternValue 即求模基数， 
defaoultNode 默认节点 
partition-pattern.txt配置 
1-32=0
33-64=1
65-96=2
97-128=3
128-256=4
配置文件中，1-32 即代表id%256后分布的范围。 如果id非数字，则会分配在defaoultNode 默认节点
```
#### 分片取舍

数据特点: 活跃的数据热度较高规模可以预期，增长量比较稳定  
![](https://github.com/YufeizhangRay/image/blob/master/Mycat/%E7%A6%BB%E6%95%A3%E5%88%86%E7%89%87.jpeg)  
  
数据特点: 活跃的数据为历史数据，热度要求不高。规模可以预期，增长量比较稳定. 优势可定时清理或者迁移数据  
![](https://github.com/YufeizhangRay/image/blob/master/Mycat/%E8%BF%9E%E7%BB%AD%E5%88%86%E7%89%87.jpeg)  
  
>1.根据业务数据的特性合理选择分片规则  
2.善用全局表、ER关系表解决join操作  
3.用好primaryKey让你的性能起飞  
  
#### 全局序列  
```
1.本地文件方式
  sequnceHandlerType = 0
	配置sequence_conf.properties
	使用next value for MYCATSEQ_XXX
2.数据库方式
	sequnceHandlerType = 1
	配置sequence_db_conf.properties
	使用next value for MYCATSEQ_XXX或者指定autoIncrement
	
3.本地时间戳方式
       ID= 64 位二进制 (42(毫秒)+5(机器 ID)+5(业务编码)+12(重复累加)
	sequnceHandlerType = 2
	配置sequence_time_conf.properties
 	指定autoIncrement

4.程序方式
	 Snowflake
	 UUID
	 Redis 
	 …
```
  
### Mycat注解  
  
```
/*!mycat: sql=select * from users where userID=1*/ select fun() from dual; 
/*!mycat: sql=select * from users where userID=1*/ CALL proc_test();
/*!mycat: sql=select * from users where userID=1*/ insert into users(id,name) select id,name from otherUsers;
/*!mycat: sql=select 1 from users*/ create table test2(id int); 
/*!mycat: db_type=slave*/ select * from employee
```
注解可以让Mycat解析本来无法解析的正常sql操作语句。注解的作用是分片路由，然后将后面的指令直接在相应的分片上执行。  
  
关联查询  
>1.用好ER表   
2.善用全局表  
3.使用注解 /*!mycat:catlet=io.mycat.catlets.ShareJoin */  
select * from users u,employee em on u.phoneNum=em.phoneNum where u.phoneNum ='13633333333' ;  
跨库查询只能支持两张物理表，并且两张物理表必须在同一个逻辑表里。  
  
### Mycat命令行监控工具  
  
• 重载配置文件  
• 查看运行状态  
• 提供性能数据 
```
mysql -uuser –ppwd -P9066 
show @@help 查看所有命令 
reload @@config
reload @@config_all
show @@database
show @@datanode
show @@datasource
show @@cache
show @@connection
show @@connection.sql
show @@backend
kill @@connection id1，id2 
show @@heartbeat
show @@sysparam
```
  
### Mycat弱XA事务机制  
  
![](https://github.com/YufeizhangRay/image/blob/master/Mycat/%E4%BA%8C%E9%98%B6%E6%AE%B5%E6%8F%90%E4%BA%A4.jpeg)  
  
为什么2PC提交.
>1是2PC才会有事务管理器统一管理的机会；  
2尽可能晚地提交事务，让事务在提交前尽可能地完成所有能完成的工作，这样，最后的提交阶段将是耗时极短，耗时极短意味着操作失败的可能性也就降低。  
  
XA 是一个两阶段提交协议，规定事务协调/管理器和资源管理器接口。  
  
二阶段提交协议为了保证事务的一致性，不管是事务管理器还是各个资源管理器，每执行一步操作，都会记录日志，为出现故障后的恢复准备依据。  
  
Mycat 第二阶段的提交没有做相关日志的记录，所以说他是一个弱XA的分布式事务解决方案。  
  
### Mycat节点扩容  
  
1.自带的mycat工具  
```
mycat 所在环境安装 mysql 客户端程序
mycat 的 lib 目录下添加 mysql 的 jdbc 驱动包
对扩容缩容的表所有节点数据进行备份，以便迁移失败后的数据恢复
编辑newSchema.xml 和 newRule.xml
配置conf/migrateTables.properties
修改bin/dataMigrate.sh，执行dataMigrate.sh
```
注意前方坑位【坑坑坑】：
>1.一旦执行数据是不可逆的  
2.只能支持分片表的扩缩容  
3.分片规则必须一致，只能节点扩或者缩  
  
2.mysqldump方式  
![](https://github.com/YufeizhangRay/image/blob/master/Mycat/%E6%95%B0%E6%8D%AE%E5%AF%BC%E5%85%A5.jpeg)  
  
```
导出数据
mysqldump -uroot -p123456 -h192.168.8.137 -c db_user_old users > users.sql
ER子表
mysqldump -uroot -p123456 -h192.168.8.137 -c --skip-extended-insert db_user_old user_address > userAddress.sql
导入
mysql -h192.168.8.151 -uroot -p123456 -P8066  -f db_user < users.sql
mysql -h192.168.8.151 -uroot -p123456 -P8066  -f db_user < userAddress.sql
```
  
### Mycat高可用  
  
![](https://github.com/YufeizhangRay/image/blob/master/Mycat/Mycat%E5%8D%95%E7%82%B9.jpeg)  
Mycat单点的情况下一旦宕机则会导致故障，于是需要多个Mycat同时使用，并用HAproxy作为负载均衡。  
  
HAproxy可以提供四层和七层的负载均衡功能，这里我们使用四层负载。  
  
四层负载均衡  
四层负载均衡也称为四层交换机，它主要是通过分析IP层及TCP/UDP层的流量实现的基于IP加端口的负载均衡  

七层负载均衡   
七层负载均衡器也称为七层交换机，位于OSI（ Open System Interconnection ，开放式系统互联）的最高层，即应用层，此时负载均衡器支持多种应用协议，常见的有HTTP、FTP、SMTP等。  
  
![](https://github.com/YufeizhangRay/image/blob/master/Mycat/Haproxy%E5%8D%95%E7%82%B9.jpeg)  
但是单个HAproxy本身也会存在单点问题。于是我们可以引入keepalived来使HAproxy变为集群。  
  
![](https://github.com/YufeizhangRay/image/blob/master/Mycat/%E9%AB%98%E5%8F%AF%E7%94%A8.jpeg)  
  
#### Mycat高可用方案Haproxy安装和配置  
Haproxy  
```
1.下载tar.gz包  
http://www.haproxy.org/download/1.8/src/haproxy-1.8.12.tar.gz  
2.解压并安装haproxy  
解压  
tar -zxvf haproxy-1.8.12.tar.gz  
安装  
cd haproxy-1.8.12  
将haproxy应用程序安装在/usr/local/haproxy 下
make TARGET=linux26 PREFIX=/usr/local/haproxy ARCH=x86_64  
make install PREFIX=/usr/local/haproxy
3.配置haproxy
	创建配置文件
	vi /usr/local/haproxy/haproxy.cfg 并编辑文件
	内容如下：
	global
		log         127.0.0.1 local2
		pidfile     /var/run/haproxy.pid
		maxconn     4000
		daemon
	defaults
		log global
		option dontlognull
		retries 3
		option redispatch
		maxconn 2000
		timeout connect 5000
		timeout client 50000
		timeout server 50000

	listen admin_status
		bind 0.0.0.0:1080 
		stats uri /admin ##haproxy自带的管理页面通过http://ip:port/admin访问
		stats auth admin:admin  ##管理页面的用户名和密码
		mode http
		option httplog

	listen allmycat_service
		bind 0.0.0.0:8096 ##转发到 mycat 的 8066 端口，即 mycat 的服务端口
		mode tcp
		option tcplog
		option httpchk OPTIONS * HTTP/1.1\r\nHost:\ www
		balance roundrobin
		server mycat_151 192.168.8.151:8066 check port 48700 inter 5s rise 2 fall 3
		server mycat_35 192.168.8.35:8066 check port 48700 inter 5s rise 2 fall 3
		timeout server 20000
		
	listen allmycat_admin
		bind 0.0.0.0:8097 ##转发到 mycat 的 9066 端口，即 mycat 的管理控制台端口
		mode tcp
		option tcplog
		option httpchk OPTIONS * HTTP/1.1\r\nHost:\ www
		balance roundrobin
		server mycat_151 192.168.8.151:9066 check port 48700 inter 5s rise 2 fall 3
		server mycat_35 192.168.8.35:9066 check port 48700 inter 5s rise 2 fall 3
		timeout server 20000

4.配置haproxy的日志输出
	haproxy采用rsyslog的方式进行日志配置
	首先安装rsyslog，可通过rpm -qa|grep rsyslog 命令判断有没有安装，没有安装自行安装（yum 方式简单明了）
	find / -name 'rsyslog.conf' 找到rsyslog的配置文件
	vi rsyslog.conf
	将#$ModLoad imudp   
	  #$UDPServerRun 514  注释解开
	找到 Save boot messages also to boot.log 在这一行下面加入
	local2.*  /var/log/haproxy.log
	保存退出
	重启rsyslog    service rsyslog restart
	
5.启动haproxy   /usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg
	此时出现：
		proxy allmycat_service has no server available!
		proxy allmycat_admin has no server available!
	因为他的check方案没有通过
		option httpchk OPTIONS * HTTP/1.1\r\nHost:\ www
		check port 48700 inter 5s rise 2 fall 3
	上面的配置的意思是，通过http检测的方式进行服务的检测，5s检测一次。
```
Xinetd  
```
通过Xinetd 提供48700端口的http服务来让haproxy进行服务的检测
在mycat的服务器上安装xinetd，
安装命令：yum install xinetd
	
找到xinetd的配置文件 cat /etc/xinetd.conf 找到 includedir /etc/xinetd.d  
	进入该目录cd /etc/xinetd.d  
	并新建 mycat_status  shell脚本，vim mycat_status
	内容如下：
	service mycat_status   #代表被托管服务的名称
	{
		flags = REUSE
		socket_type = stream			# socket连接方式
		port = 48700				# 服务监听的端口
		wait = no				# 是否并发
		user = root				# 以什么用户进行启动
		server =/usr/local/bin/mycat_status     # 被托管服务的启动脚本
		log_on_failure += USERID 		# 设置失败时，UID添加到系统登记表
		disable = no       			#是否禁用托管服务，no表示开启托管服务
	}
	保存退出
	
创建托管服务启动脚本/usr/local/bin/mycat_status
	vim /usr/local/bin/mycat_status
	内容如下:
	mycat=`/root/mycat/bin/mycat status |grep 'not running'| wc -l`
	if [ "$mycat" = "0" ];
	then
		/bin/echo -e "HTTP/1.1 200 OK\r\n"
	else
		/bin/echo -e "HTTP/1.1 503 Service Unavailable\r\n"
	fi	
        保存退出 并赋予执行权限chmod +x /usr/local/bin/mycat_status
	验证脚本的正确性 sh /usr/local/bin/mycat_status  如果返回 200OK字样说明成功
	
加入mycat_status服务
	vi /etc/services
	在末尾加入以下内容：
	mycat_status 48700/tcp # mycat_status
	保存退出，重启xinetd服务，  service xinetd restart
	
验证服务是否启动成功
	netstat -antup|grep 48700

此时重启haproxy服务即正常
```

------------------------以上为-haproxy 4层代理的负载均衡配置-----------------------  
---------------------下面内容为keepalive+haproxy的高可用方案----------------------  
  
keepalive  
```
keepalive安装  （192.168.8.35 -> MASTER   192.168.8.151 -> BACKUP ） 
	yum install  keepalived
	find / -name 'keepalived.conf'
	vi /etc/keepalived/keepalived.conf
	内容如下
	! Configuration File for keepalived
	vrrp_instance VI_1 {
		state MASTER            #192.168.8.151 上改为 BACKUP
		interface ens33         #对外提供服务的网络接口
		virtual_router_id 100   #VRRP 组名，两个节点的设置必须一样，以指明各个节点属于同一 VRRP 组
		priority 150            #数值愈大，优先级越高,192.168.8.151 上改为比150小的正整数
		advert_int 1            #同步通知间隔
		authentication {        #包含验证类型和验证密码。类型主要有 PASS、AH 两种，通常使用的类型为 PASS，据说AH 使用时有问题
			auth_type PASS
			auth_pass 1111
		}
		virtual_ipaddress { #vip 地址    ens33  通过ifconfig获取
			192.168.8.233 dev ens33 scope global
		}
	} 
	编辑保存
	
启动keepalived，执行命令 service keepalived start
	
应用通过访问192.168.8.233 即访问抢占了vip（192.168.8.233）的物理机（192.168.8.35）。
通过上面的配置，我们可以将前端的请求转到抢占了vip（192.168.8.233）的物理机（192.168.8.35）。
但是没有通过监听haproxy的服务，或者说我们没有根据haproxy服务来进行vip的降级。
keepalived提供了很多的配置来做服务的检测和降级，但是我们今天不学keepalived的方式我们采用一种定时任务（linux自带的crontab）的方式来做。
	
1.创建check_haproxy.sh 并编辑内容  vi /root/script/check_haproxy.sh
	内容如下：
	#!/bin/bash
	LOGFILE='/root/log/checkHaproxy.log'
	date >> $LOGFILE
	count=`ps aux | grep -v grep | grep /usr/local/haproxy/sbin/haproxy | wc -l`
	if [ $count = 0 ];
	then
		echo 'first check fail , restart haproxy !' >> $LOGFILE
		/usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg
	else
		exit 0
	fi   
        sleep 3
	count=`ps aux | grep -v grep | grep /usr/local/haproxy/sbin/haproxy | wc -l`
	if [ $count = 0 ];
	then
		echo 'second check fail , stop keepalive service !' >> $LOGFILE
		service keepalived stop
	else
		echo 'second check success , start keepalive service !' >> $LOGFILE
		keepalived=` ps aux | grep -v grep | grep /usr/sbin/keepalived | wc -l`
		if [ $count = 0 ];
		then
			service keepalived start
		fi
		exit 0
	fi
	
2.执行 crontab -e 编辑定时任务 每一分钟检测haproxy服务存活，如果服务启动不了，停掉keepalived服务，vip即转发至backup的192.168.8.151的机器
	* * * * * sh /root/script/check_haproxy.sh	 
      分 时 日 月 周
```
	
[返回顶部](#mycat-practice)
