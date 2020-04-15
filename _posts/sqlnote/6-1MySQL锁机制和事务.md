# MySQL锁机制和事务

## InnoDB锁机制

- InnoDB存储引擎支持行级锁，其大类可细分为共享锁和排它锁两类

  - 共享锁（S）: 允许拥有共享锁的事务读取改行数据。当一个事务拥有一行的共享锁时，另外的事务可以在同一行数据也获得共享锁，但另外的事务无法获得同一行数据上的排它锁。
  - 排它锁（X）: 允许拥有排它锁的事务修改或者删除该行数据。当一个事务拥有一行的排它锁时，另外的事务在此行数据上无法获得共享锁和排它锁，只能等待第一个事务的锁释放。

- 除了共享锁和排它锁，InnoDB也支持意图锁。该锁类型是属于表级锁，表名事务在后期会对该表的行施加共享锁或者排它锁。所以意图锁也有两类。

  - 共享意图锁（IS）：事务将会对表的行施加共享锁
  - 排他共享锁（IX）:  事务将会对表的行施加排它锁
  - 举例来说select...for share mode语句就是施加了共享意图锁，而select...for update语句就是施加了排他意图锁。

- 这四种锁之间的相互共存和排斥关系如下：

  

|      |    X     |     IX     |     S      |     IS     |
| :--: | :------: | :--------: | :--------: | :--------: |
|  X   | Confict  |  Conflict  |  Conflict  |  Conflict  |
|  IX  | Conflict | Compatible |  Conflict  | Compatible |
|  S   | Conflict |  Conflict  | Compatible | Compatible |
|  IX  | Conflict | Compatible | Compatible | Compatible |

- 所以决定一个数据请求为数据加锁时能否立即施加上锁，取决于该行数据上已经存在的锁时候和请求的锁可以共存还是排斥关系。当相互之间是可以共存时则立即施加锁，当相互之间是排斥关系时则需要等待已经存在的锁被释放才能施加。



##  INnoDB锁相关系统表

- information_schema.INNODB_TRX 记录了InnoDB中每一个正在执行的事务，包括该事务获得的锁信息，事务开始时间、事务是否在等待锁。。等信息

  ![](https://tva1.sinaimg.cn/large/00831rSTly1gdfxpvedd4j31860u0b2b.jpg)

  - 查看mysql innodb中每一个正在执行的事务

    ```SQl
    mysql> select * from information_schema.innodb_trx\G
    *************************** 1. row ***************************
                        trx_id: 112913
                     trx_state: LOCK WAIT
                   trx_started: 2020-04-02 00:15:37
         trx_requested_lock_id: 112913:340:3:4
              trx_wait_started: 2020-04-02 00:19:10
                    trx_weight: 2
           trx_mysql_thread_id: 8
                     trx_query: update students2 set last_name='wto' where sid=66
           trx_operation_state: starting index read
             trx_tables_in_use: 1
             trx_tables_locked: 1
              trx_lock_structs: 2
         trx_lock_memory_bytes: 1184
               trx_rows_locked: 1
             trx_rows_modified: 0
       trx_concurrency_tickets: 0
           trx_isolation_level: REPEATABLE READ
             trx_unique_checks: 1
        trx_foreign_key_checks: 1
    trx_last_foreign_key_error: NULL
     trx_adaptive_hash_latched: 0
     trx_adaptive_hash_timeout: 10000
              trx_is_read_only: 0
    trx_autocommit_non_locking: 0
    *************************** 2. row ***************************
                        trx_id: 112912
                     trx_state: RUNNING
                   trx_started: 2020-04-02 00:15:27
         trx_requested_lock_id: NULL
              trx_wait_started: NULL
                    trx_weight: 3
           trx_mysql_thread_id: 10
                     trx_query: NULL
           trx_operation_state: NULL
             trx_tables_in_use: 0
             trx_tables_locked: 0
              trx_lock_structs: 2
         trx_lock_memory_bytes: 360
               trx_rows_locked: 3
             trx_rows_modified: 1
       trx_concurrency_tickets: 0
           trx_isolation_level: REPEATABLE READ
             trx_unique_checks: 1
        trx_foreign_key_checks: 1
    trx_last_foreign_key_error: NULL
     trx_adaptive_hash_latched: 0
     trx_adaptive_hash_timeout: 10000
              trx_is_read_only: 0
    ```

  

- performance_schema.data_locks  记录了innodb中事务的每个锁信息，以及当前事务的锁正在组织其他事务获得锁。（5.7以后采用的）

  ![](https://tva1.sinaimg.cn/large/00831rSTly1gdfxrxt6obj30pf0dfdi0.jpg)



- sys.innodb_lock_waits记录了innodb中事务之间相互等待锁的信息

  ![](https://tva1.sinaimg.cn/large/00831rSTly1gdfxu5z9puj30ot0d5752.jpg)



### Innodb行级锁

行级锁是施加在索引行数据上的锁，比如`select c1 from t where c1=10 for update`语句是在t.c1=10的索引行上增加锁，来阻止其他事务对对应索引进行的insert、uodate、delete操作

当一个innodb表没有任何索引时，则行级锁会施加在隐含创建的聚簇索引上，所以说当一条sql没有走任何索引时，那么将会在每一条聚簇索引后面加X锁，这个类似于表锁，但原理上和表锁应该是完全不同的。



- 查询已执行未提交的事务 执行的是什么语句

```SQL
 select * from events_statements_current\G
# 可以查到thiredID

select thread_id,processlist_id from threads;
# 查询进程与线程对应关系

select * from sys.innodb_lock_waits\G  # 出现于mysql5.7之后

```



### 间隔锁

- 当我们用范围条件而不是相等条件检索数据，并请求共享或排它锁时，innodb会给符合条件的已有数据记录的索引项加锁；对于键值在条件范围内但并不存在的记录，叫做“间隙”（GAP）,innodb也会对这个“间隙”加锁
- 间隔锁是施加在记录之间的间隔上的锁，锁定一个范围的记录，但不包括记录本身，比如`select c1 from t where c1 between 10 and 20 for update `语句，尽管有可能对C1字段来说当前表里没有c=15的值，但是还是会阻止c=15的数据的插入操作，是因为间隔锁已经把索引查询范围内的间隔数据也都锁住了
- 间隔锁的使用只在部分事务隔离级别才是生效的
- 间隔锁只会组织其他事务的插入操作



- gap lock的前置条件
  1. 事务隔离级别为REPEATABLE-READ,且sql走的索引为非唯一索引（无论是等值还是范围检索）
  2. 事务隔离级别为REPEATABLE-READ,且sql是一个范围的当前读操作，这是即使是唯一索引也会加gap lock。
- innodb_locks_unsafe_for_binlig(强制不使用间隔锁)参数，后续在8.0版本中取消



###  Next-key锁

在默认情况下，mysql的事务隔离级别是可重复读，默认采用next-key locks。next-key locks就是记录锁和间隔锁的结合，即除了锁住记录本身，还要再锁住索引之间的间隙。



###  InnoDB锁相关系统变量



```sql
# 查看自动提交状态
mysql> show variables like '%autocommit%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | ON    |
+---------------+-------+
1 row in set (0.01 sec)

# 查看锁等待的超时时间
mysql> show variables like 'innodb_lock_wait_timeout';
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| innodb_lock_wait_timeout | 7200  |
+--------------------------+-------+
1 row in set (0.01 sec)


```





select 默认不加锁，可以自行加锁

```SQL
select * from tb1 lock in share mode;   # 为select 添加共享锁

select * from students for update;     # 为select 添加排它锁
rollback； # 解锁



```



```SQL
mysql> show variables like '%iso%';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| tx_isolation  | REPEATABLE-READ |
+---------------+-----------------+
1 row in set (0.00 sec)

```



## InnoDB事务级别隔离

- InnoDB存储引擎提供了四种事务隔离级别，分别是
  - READ UNCOMMITTED： 读取未提交内容
  - READ COMMITTED：读取提交内容
  - REPEATABLE READ:  可重复读，默认值
  - SERIALIZABLE:   串行化



| 事务隔离级别                 | 脏读 | 不可重复读 | 幻读 |
| ---------------------------- | ---- | ---------- | ---- |
| 读未提交（read-uncommitted） | 是   | 是         | 是   |
| 不可重复读（read-committed） | 否   | 是         | 是   |
| 可重复读（repeatable-read）  | 否   | 否         | 是   |
| 串行化（serializable）       | 否   | 否         | 否   |



- 事务隔离级别参数与设置
  - 可通过`--tx-isolation`or `--transaction-isolation`参数设置实例级别的事务隔离级别
  - 也可通过` set [session|global] transcation isolation level ` [ READ UNCOMMINTED | READ COMMINTED | repeatable-read | serializable ] 语句修改当前数据库连接或者是后续创建的所有数据库连接的事务隔离级别



- repeatable-read： 可重复读，默认值，是表名对同一个事务来说，第一次读取数据时会创建快照，在事务结束前的其他读操作（不加锁）会获得和第一次读相同的结果。当读操作是加锁的读语句（select...for update或lock in share mode），或者是update和delete语句时，加锁的方式依赖于语句是否使用唯一索引访问唯一值或者范围值。
  - 当访问的是唯一索引的唯一值时，则InnoDB会在索引行上施加行锁
  - 当访问唯一索引的范围值时，则会在扫描的索引行上增加间隔锁或者next-key锁以防止其他链接对此范围的插入。
- read committed：读取提交内容，意味着每次读都会有自己最新的快照。对于加锁读语句或者update和delete语句会在对应的行索引上增加锁，但不像可重复读一样会增加间隔锁，因此其他的事务的执行插入操作时如果是插入非索引行上的数值，则不影响插入。
  - 该隔离级别是禁用间隔锁的，所以会导致幻读的情况
  - 如狗是使用此隔离级别，就必须使用行级别的二进制日志
  - 此隔离级别另外的特点：
    - 1. 对于uodate和delete语句只会在约束条件对应的行上增加锁
      2. 对于update语句来说，如果对应的行上已经有锁，则InnoDB会执行半一致读的操作，来确定update语句对应的行在上次commit之后的数据是否在锁的范围。如果不是，则不影响update操作，如果是则需要等待对应的锁解开。
- read uncommitted: 读取未提交内容，所读到的数据可能是脏数据
- seaializable：串行化，此隔离级别更接近于可重复读这个级别。只是当autocommit功能被禁用后，InnoDB引擎会将每个select语句隐含的转化为select...lock in share mode。







