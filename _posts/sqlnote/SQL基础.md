# SQL基础

##  约束条件

### 列的约束及属性

```
promary key：主键约束，同事保证唯一性和非空，每个表只能有一个PK，建议业务无关列
foregin key：外键约束，用于限制连个表的关系，保证从表该字段的值来自于主表相关联的字段的值
not full： 非空约束，保证字段的值不能为空
default：默认约束，保证字段总会有值，即使没有插入值，都会有默认值
unique： 唯一，保证唯一性但是可以为空，比如手机号
unsigned： 无符号
comment： 注释
```



## SQL技术讲解DDL

### DDL基础应用

#### 库定义

```sql
# 建库
create database db01 if not exists charset utf8mb4;
show wanrings;

# 查库
show databases;
show create database db01;
```



#### 建表规范

```
表名：不要大写字母，不要数字开通，不要超过18字符，不要用内置字符串，和业务有关
列明：业务有关，不要内置字符，不要超过18字符
数据类型：合格的，精简的，完整的
每个表必须要有且主键。每个列尽量not null，尽量不要使用外键
每列要有注释
存储引擎innoDB，字符集utf8mb4
```



#### 查表

```sql
show tables；
show create tables;

#查看表的定义信息
select table_name,column_name,data_type,column_comment from information_schema.columns where table_schema='table_name'
```



### OnlineDDL

 ```
建议：
· 一般DDL操作最好都采用pt-osc或gh-ost这样的工具来实施，并且实施之前务必要先检查当前目标表上是否有重大事务或者大查询未结束，避免严总的MDL锁（Meatdata Lock）等待。
· 除了8.0以上版本，除了追加式新增列、表改名、新增虚拟列这三张支持INSTANT的操作可以直接跑DDL,其余的都统统采用pt-osc或gh-ost工具，这样更不容易出问题
· 执行ALERT TABLE DDL时，不要执行ALGORITHM=?,LOCK=?选项，因为MYSQL会自行判断该采用哪种方式。本来可以INPLACE的，可能不小心指定成COPY就悲剧了。
 ```



### DML语句基础

#### insert语句



#### update语句



#### select 语句

```
多子句的执行顺序
select
	from 从哪里来
	where 过滤条件
	group by 分组条件
		select_list 列条件
	having 后过滤条件
	order by 排序条件
	limit 分页
```



##### 多表连接

```sql
select * from students inner join dept; # 内连接 笛卡尔积，显示两张表所有行相互的拼接结果
select * from students inner join dept on students.dept_id=dept.id ;  # 在on后加条件，只显示关联行

# 上面语句的另一种写法
select * from students,dept where students.dept_id=dept.id
# inner join 后的条件筛选
select * from students inner join dept on students.dept_id=dept.id where sid>1;
```



**别名应用**

```
表别名可在任意子句中调用
列别名只可在 having和order by子句中调用
```



**join多表连接工作逻辑和查询优化思路**

```SQL
--- 多表连接工作逻辑
1. 优化器，在内连接中，选择驱动表
	（1）多张表时，从左至右，查看表中的索引情况，有索引选为驱动表
	（2）如果第一张表没有索引，查看下一张。直到找到有索引的表作为驱动表

2. 多表连接算法
SNLJ
BNLJ
BKAJ
show veriables like '%optimeize%'; # 查看算法

3. 优化重点
强制‘小表’为驱动表; （使用left join，可以选择左表为驱动表）
建立合适的索引
开启优化器
set globlal optimizer_switch='batched_key_access=ON,mrr_cost_based=off';

4. left join 优化 not in语句（not in不走索引）
原句：
select * from a where id not in (select bid from b);
改写
select * form a left join b on a.id=b.bid where b.bid is NULL;

```



**子查询**

```sql
select xxx from (sybquery)  # 杜绝
select xxx from  where (subquery)  # join改写
```



















