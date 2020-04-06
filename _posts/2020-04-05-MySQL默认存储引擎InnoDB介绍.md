---
layout:     post
title:      MySQL默认存储引擎InnoDB介绍
subtitle:   innodb
date:       2020-04-05
author:     hongfei
header-img: img/post-bg-debug.png
catalog: true
tags:
    - mysql
    - innodb
    
---

#  InnoDB内核

## InnoDB存储引擎介绍

MySQL从5.5版本开始将InnoDB作为默认存储引擎，该存储引擎是第一个完整支持事务ACID特性的存储引擎，且支持数据行锁、多版本并发控制(MVCC)，外键，以及一致性非锁定读。

作为默认存储引擎，也就意味着默认创建的表都会使用此存储引擎，除非使用ENGINE=参数指定创建其他存储引擎的表。

### InnoDB的关键属性

- ACID事务特性支持，包括commit，rollback以及crash恢复的能力
- 行级锁以及多版本控制MVCC
- 利用逐渐的聚簇索引(clustered index)在底层存储数据，以提升对主键查询的IO性能
- 支持外键功能，管理数据的完整性



### InnoDB和ACID模型

ACID模型是关系型数据库普遍支持的事务模型，用来保证数据的一致性，其中的ACID分别代表：

- A: atomicity原子性：事务是一个不可再分割的工作单位，事务中的操作要么都发生，要么都不发生
- C: consistency一致性：事务开始之前和事务结束之后，数据库的完整性约束没有被破坏。这是说数据库事务不能破坏关系数据的完整性以及业务逻辑上的一致性。
- I: isolation独立性：多个事务并发访问时，事务之间是隔离的，一个事务不应该影响其他事务运行效果
- D: durabilty：在事务完成之后，该事务对所有数据库所做的更改便持久保持在数据库之中，并不会被回滚。

事务的隔离性是通过MySQL的锁机制实现

原子性，一致性，持久性则通过MySQL的redo和undo日志记录来完成



###  InnoDB多版本控制

为保证并发操作和回滚操作，InnoDB会将修改改签的数据存放在回滚段中。InnoDB会在数据库的每一行上额外增加三个字段以实现多版本控制，第一个字段是DB_TBX_ID用来存放针对改行最后一次执行insert、update操作的事务ID，而delete操作也会被认为是updte，只是会有额外的一位来代表事务为删除操作；第二个字段是DB_ROLL_PTR指针指向回滚段里对应的undo日志记录；第三个字段是DB_ROW_ID代表每一行的行ID。

回滚段中的undo日志记录只有在事务commit提交之后才会被丢弃，未避免回滚段越来越大，要注意及时执行commit命令。

![](https://tva1.sinaimg.cn/large/00831rSTly1gdchpl0l1ej30zm0iok14.jpg)

- 事务执行方式：

```SQL
start transaction;
SELECT
INSERT
SET
DELETE
commit;
rollback;
```



- 事务隔离级别
  - 同一个事务中，可重复读。读到的内容一致，即使别的事务对数据进行了更改并提交



###  InnoDB体系结构

- B+树结构

![](https://tva1.sinaimg.cn/large/00831rSTly1gddjlaj6j7j30y80jgwnh.jpg)



![](https://tva1.sinaimg.cn/large/00831rSTly1gddjm73p4lj314w0jogtm.jpg)



### InnoDB数据文件存储结构

- 索引组织表（聚簇表）
- 根据表逻辑主键排序
- 数据节点每页16k - 数据页

![image-20200331235026058](/Users/wph/Library/Application Support/typora-user-images/image-20200331235026058.png)



特点：

- 根据主键寻址速度很快
- 主键值递增的insert插入效率较好
- 主键值随机insert插入操作效率差



##  存储引擎

### 存储引擎体系架构

![](https://tva1.sinaimg.cn/large/00831rSTly1gddjq04rqsj30nd0edn34.jpg)

- InnoDB存储引擎内存空间
  - 缓存池Buffer Pool  最大的空间
  - redo log buffer
  - double write
  - dynamic memeory pool ?? 



- 缓存池
  - buffer pool缓存池是innodb在内存中开辟的用来存放缓存表数据和索引数据的区域，一般可以设置为50%~80%的屋里内存大小，通过对经常访问的数据防止到内存当中来加快访问速度。
  - bufer pool以page页的格式组成，页之间组成list列表，并通过LRU算法(最近最少使用算法)，对长久不使用的页进行置换
  - 数据的读写需要经过缓存（缓存在buffer poll 即在内存中），数据以整页(16k)为单位读取到缓存中。
  - 缓存中的数据以LRU策略换出。
  - IO效率高，性能好

![](https://tva1.sinaimg.cn/large/00831rSTly1gdci4lzkgrj30dt07fgmv.jpg)



- Adaptive Hash Index（自适应哈希索引）

  - Adaptive hash index属性使得InnoDB更像是内存数据库。该属性通过innodb_adaptive_hash_index开启，也可以通过--skip-innodb_adaptive_hash_index参数关闭

  - InnoDB存储引擎会监控对表上索引的查找，如果观察到建立哈希索引可以带来速度的提升，则建立哈希索引，所以称之为**自适应**(adaptive)的。自适应哈希索引通过缓冲池的B+树构造而来，因此简历的速度很快。而且不需要将整个表都建哈希索引。InnodB存储引擎会自动根据访问的频率和模式来为某些页建立哈希索引。

  - 哈希是一种非常开IDE等值查找方法，在一般模式下这种查找的时间复杂度为O(1),即一般仅需要一次查找就能定位数据。而B+树的查找次数，取决于B+树的高度，在生产环境汇总，B+树的高度一般为3-4层，故需要3-4次的查询

  - AHI(Adaptive hash index)有一个要求，就是对这个页的连续访问模式必须是一样的。

  - 例如对于(a，b)访问模式情况：

    ```SQL
    where a =xxx
    where a=xxx and b=xxx
    ```

  - AHI启动后，读写速度提升了2倍，辅助索引的连续操作性能可以提高5倍。

  - AHI是数据库自动化的，DBA只需要知道开发人员去尽量使用符合AHI条件的查询，以提高效率。

  

- Redo_log_buffer

  - redo_log_buffer是一块引来存放写入redo log文件内容的内存区域，内存的大小由innodb_log_buffer_size参数确定。该buffer的内容会定期刷新到磁盘的redo log文件中。

  - `innodb_flush_log_at_trx_commit`  决定刷新到文件的方式

  - `innodb_flush_log_at_timeout` 决定刷新的频率

    ```
    innodb_flush_log_at_trx_commit
    0：每秒写入并持久化一次（不安全，性能高，无论mysql或者服务器宕机，都会丢最多一秒的数据）
    1：每次commit都持久化（安全，性能低，IO负担重）
    2：每次commit都写入内存的聂村缓存，每秒再刷新到磁盘（安全，性能折中。mysql重启数据不会丢失，服务器宕机最多丢失一秒的数据）
    
    innodb_flush_log_at_timeout 
    这个参数决定最多丢失多少秒的数据，默认是1秒。
    ```

- 系统表空间

  - innodb的系统表空间用来存放表和索引数据，同事也是doublewrite缓存，change缓存和回滚日志的存储空间，系统表空间是被多个表共享的表空间。
  - 默认情况下，系统表孔教只有一个系统数据文件，名为ibdata1。系统数据文件的位置和个数由参数`innodb_data_file_path`决定

为了IO效率，数据库修改的文件都在内存缓存中完成；那一旦断电，内存中的数据小时，数据库是如何保证数据的完整性？那就是数据持久化与事务日志。

如果宕机了，则应用已经持久化好了的日志文件，读取日志文件中没有被持久化到数据记录文件里面的记录；将这些记录重新持久化到我们的数据文件中。

![](https://tva1.sinaimg.cn/large/00831rSTly1gdci96telij30dh0cttct.jpg)



![](https://tva1.sinaimg.cn/large/00831rSTly1gdci9fc9onj30f5081mxr.jpg)





- doublewrite缓存
  - 是位于系统表空间的存储区域，用来缓存innodb的数据也从innodb的数据页从`innodb buffer poll`中flush之后并写入到数据文件之前，所以当操作系统或者数据库进程在数据页写磁盘的过程中崩溃，innodb可以在doublewrite缓存中找到数据页的备份而用来执行creash恢复。
  - 数据也写入doublewrite缓存的动作所需要的IO消耗要小雨写入到数据文件的消耗，因为此写入操作会以一次大的连续快的方式写入。



![](https://tva1.sinaimg.cn/large/00831rSTly1gddiy299w1j30kv0dltea.jpg)







![](https://tva1.sinaimg.cn/large/00831rSTly1gddiyntwiaj30g30a440a.jpg)

数据脏页构成1M的块，直接copy



![](https://tva1.sinaimg.cn/large/00831rSTly1gddj30wdxej30kr07wdma.jpg)



![](https://tva1.sinaimg.cn/large/00831rSTly1gddj37ripkj30kl07479w.jpg)



![](https://tva1.sinaimg.cn/large/00831rSTly1gddj4cqdtrj30km05hdhg.jpg)



- 数据文件
  - db_name/tbl_name.idb 存放表的实际数据
  - Ibdata1 共享表数据，表的元数据信息
  - ib_logfile0/1 记录redo-log日志文件
  - Ibtmp1 临时表空间文件
  - binlog 二进制日志，记录所有增删改操作



![](https://tva1.sinaimg.cn/large/00831rSTly1gddja8ylssj30kk0ann1h.jpg)

独立表空间，是从5.7开始的。

`show variables like '%pre_tabale%'`

innodb_file_per_table 每张表一个数据文件



![](https://tva1.sinaimg.cn/large/00831rSTly1gddjcmg2qfj30ko05in05.jpg)

如果这两个文件的修改时间比较接近，说明这个文件的大小不太够用



![](https://segmentfault.com/img/bVbw1jB?w=700&h=538)





##  InnoDB存储引擎配置

### 启动配置文件

- 启动配置

InnoDb合理的规范方式是在创建数据库实例之前就定义好数据文件，日志文件和数据页大小等相关属性

- 指定配置文件

MySQL实例启动需要依赖my.cnf配置文件，而配置文件可以存在与多个操作系统目录下。my.cnf文件的默认查找路径，从上倒下找到的文件先读，但优先级逐级提升。

![](https://tva1.sinaimg.cn/large/00831rSTly1gddk3rek8hj30rg0aon5i.jpg)

```SQL
mysql> show variables like '%port%';
+---------------------+-------+
| Variable_name       | Value |
+---------------------+-------+
| innodb_support_xa   | ON    |
| large_files_support | ON    |
| port                | 3306  |
| report_host         |       |
| report_password     |       |
| report_port         | 3306  |
| report_user         |       |
+---------------------+-------+
7 rows in set (0.01 sec)
```



![](https://tva1.sinaimg.cn/large/00831rSTly1gddkkujjkaj30iw09b442.jpg)



```SQl
# ibdata1 的设置
mysql> show variables like '%innodb_data%';
+-----------------------+------------------------+
| Variable_name         | Value                  |
+-----------------------+------------------------+
| innodb_data_file_path | ibdata1:12M:autoextend |
| innodb_data_home_dir  |                        |
+-----------------------+------------------------+
2 rows in set (0.00 sec)
```

ibdata1:12M:autoextend   初始是12M ，自动增加。配置文件中能指定的大小是当前配置文件的实际大小，但是不能自定义指定这个文件的大小。



```SQl
mysql> show variables like '%autoextend%';  # 文件自动扩展的频率
+-----------------------------+-------+
| Variable_name               | Value |
+-----------------------------+-------+
| innodb_autoextend_increment | 64    |
+-----------------------------+-------+
1 row in set (0.00 sec)
```



![](https://tva1.sinaimg.cn/large/00831rSTly1gddknlxmyzj30je0cgdjg.jpg)

指定两个ibdata文件时，只需要指定一次就够了。配置两个文件后，只需要指定第二个文件的自动扩展。



###  日志文件配置

默认情况先innodb会在数据文件夹下创建两个48M的日志文件，分别是ib_logfile0和ib_logfile1.

innodb_log_group_home_dir 参数用来定义redo日志的文件位置。

![](https://tva1.sinaimg.cn/large/00831rSTly1gddkuogzvnj30p20byq8j.jpg)



```SQl
mysql> show variables like '%innodb_log_file_size%';
+----------------------+----------+
| Variable_name        | Value    |
+----------------------+----------+
| innodb_log_file_size | 50331648 |
+----------------------+----------+
1 row in set (0.00 sec)
```



![](https://tva1.sinaimg.cn/large/00831rSTly1gddkx04nzrj30pa0cq0wg.jpg)



###  数据页配置

InnoDB_page_size参数用来指定所有innodb表空间的数据页的大小，默认是16k大小，也可以设置为64k，32k, 8k和4k，一般设置为存储次哦按的block size接近的大小。



###  内存相关配置

InnoDB_buffer_pool_size参数确定了缓存表数据和索引数据的内存区域大小，默认为128M，推荐谁知为系统内存的50%~80%。在服务内有大量内存的情况下，也可以设置多个缓存以提高系统并发度。

InnoDB_buffer_pool_instances参数就是用来做这个设置。

Innodb_log_buffer_size参数确定了redo log缓存的大小，默认值是16M，其大小取决于是否有某些大的事务会大量修改数据而导致在事务执行过程中就要写日志文件。



###  InnoDB只读设置

![](https://tva1.sinaimg.cn/large/00831rSTly1gddkyzfvojj30ma06awfo.jpg)



###  InnoDB buffer pool设置



![](https://tva1.sinaimg.cn/large/00831rSTly1gddkz7y5raj30p7073n2j.jpg)



![](https://tva1.sinaimg.cn/large/00831rSTly1gddl2gwa7jj30os0c8q8s.jpg)



![](https://tva1.sinaimg.cn/large/00831rSTly1gddla0m8h2j30j408otew.jpg)





![](https://tva1.sinaimg.cn/large/00831rSTly1gddld91d4ij30ir06ujvy.jpg)

第一次触发flush  是数据比例达到10%。 强烈的flush





![](https://tva1.sinaimg.cn/large/00831rSTly1gddlf5zo5yj30jc0am42q.jpg)



```SQl
show variables like '%buffer%';

| innodb_buffer_pool_dump_at_shutdown | OFF            |
| innodb_buffer_pool_load_at_startup  | OFF            |
```



了解 change buffer :

![](https://tva1.sinaimg.cn/large/00831rSTly1gddlkcgkp0j30jf09sae0.jpg)



### InnoDB其他配置

mysql相关现成是联系内存区和硬盘区的枢纽

```SQl
# 查看所有线程
mysql> use performance_schema;
mysql> select * from threads;
```



![](https://tva1.sinaimg.cn/large/00831rSTly1gddlr8y2v3j30jb0ap0x1.jpg)



![](https://tva1.sinaimg.cn/large/00831rSTly1gddlrp7ao4j30j506z40t.jpg)



![](https://tva1.sinaimg.cn/large/00831rSTly1gddlsfu2ucj30jk09vdjz.jpg)



![](https://tva1.sinaimg.cn/large/00831rSTly1gddlu1dtdrj30jd0aswgt.jpg)



![](https://tva1.sinaimg.cn/large/00831rSTly1gddmlbnw6kj30h5098wfo.jpg)



![](https://tva1.sinaimg.cn/large/00831rSTly1gddmn22e8lj30jn0arn0t.jpg)



![](https://tva1.sinaimg.cn/large/00831rSTly1gddmp6xg8vj30b409p403.jpg)

区别度。区别度越高，筛选越有利。





- 配置单表数据的文件表空间

InnoDB的单表数据文件表空间戴白欧美歌InnoDB表的数据和索引数据都存放在单独的.idb数据文件中，每个.idb数据文件代表独立的表空间。此属性通过innodb_file_pre_table配置。5.7以后的特性 。

此配置的主要优势：

1. 当删除表或者truncate表的时候，意味着对磁盘空间可以回收。而共享表空间时删除一个表时空间不会释放而只是文件里有空闲空间。
2. truncate table命令要比共享表空间快
3. 通过定义create table ... data directory='文件绝对路径'，可以将特定的表放在特定的磁盘或者存储空间
4. 可以将单独的表屋里拷贝到另外的MySQL实例中



此配置的劣势：

1. 每个表都有未使用的空间，意味着磁盘空间有些浪费。

