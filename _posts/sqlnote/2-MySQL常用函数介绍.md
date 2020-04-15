

# MySQL常用函数介绍

### mysql操作符

[mysql操作符](https://www.runoob.com/mysql/mysql-operator.html)

## 操作符优先级

MySQL中所有运算符的优先级的顺序按照从高到低，从上到下，依次降低。一般情况下，级别高的运算符先进行计算，如果级别相同，MySQL按照表达式的顺序从左到右依次计算。

```SQL
优先级 运算符
(最高)  !
    -（负号）,~（按位取反）
    ^（按位异或）
    *,/(DIV),%(MOD)
    +,-
    >>,<<
    &
    |
    =(比较运算),<=>,<,<=,>,>=,!=,<>,IN,IS NULL,LIKE,REGEXP
   BETWEEN AND,CASE,WHEN,THEN,ELSE
   NOT
   &&,AND
   XOR
   ||,OR
(最低)    =(赋值运算),:=
```



```SQL
mysql> select 1+2*3;
+-------+
| 1+2*3 |
+-------+
|     7 |
+-------+
1 row in set (0.00 sec)

mysql> select (1+2)*3;
+---------+
| (1+2)*3 |
+---------+
|       9 |
+---------+
1 row in set (0.00 sec)
```





###对比操作符

![](https://tva1.sinaimg.cn/large/00831rSTly1gd2rj444roj30hz0aoadl.jpg)



![](https://tva1.sinaimg.cn/large/00831rSTly1gd2rnfjq6dj30ku0b5whk.jpg)



![](https://tva1.sinaimg.cn/large/00831rSTly1gd2rtvfhdjj30gf053750.jpg)



![](https://tva1.sinaimg.cn/large/00831rSTly1gd4fqgkzhtj30oe0k4adl.jpg)

![](https://tva1.sinaimg.cn/large/00831rSTly1gd2ry00dffj30la09swgq.jpg)



![](https://tva1.sinaimg.cn/large/00831rSTly1gd2ryhlqqyj30he0afmz0.jpg)



### 逻辑操作符

![](https://tva1.sinaimg.cn/large/00831rSTly1gd4fqxmii1j30to08qjvm.jpg)



### 分配操作符

![](https://tva1.sinaimg.cn/large/00831rSTly1gd2sav9tj5j30l50bndgr.jpg)

```SQL
mysql> set @a=1;
Query OK, 0 rows affected (0.01 sec)

mysql> select @a;
+------+
| @a   |
+------+
|    1 |
+------+
1 row in set (0.00 sec)

mysql> select @b;
+------+
| @b   |
+------+
| NULL |
+------+
1 row in set (0.00 sec)

+----------+
| count(*) |
+----------+
|        6 |
+----------+
1 row in set (0.00 sec)

# select 语句中使用 := 赋值
mysql> select @b:=count(*) from user;
+--------------+
| @b:=count(*) |
+--------------+
|            6 |
+--------------+
1 row in set (0.00 sec)

mysql> select @b;
+------+
| @b   |
+------+
|    6 |
+------+
1 row in set (0.00 sec)
```





## 流程控制函数

![](https://tva1.sinaimg.cn/large/00831rSTly1gd2sj9ghrpj30ei05iwfx.jpg)

### case when 

```SQL
# CASE columu_name when VALUES then XXX END 
# 替换内容
SELECT SID,SNAME,GENDER, DEPT_ID, CASE DEPT_ID WHEN 1THEN 'COMPUTER SCIENCE' WHEN 2 THEN 'DEUCATION' AS dept_name END FROM STUDENTS;

# case when columu=? ... end
SELECT SID,SNAME,GENDER, DEPT_ID, CASE  WHEN DEPT_ID=1 THEN 'COMPUTER SCIENCE' WHEN DEPT_ID>2 THEN 'DEUCATION' AS dept_name END FROM STUDENTS;
```



### if

![](https://tva1.sinaimg.cn/large/00831rSTly1gd2sy1rxnyj30dx0budig.jpg)



```SQL
# IF
mysql> SELECT IF(1>2, 2 ,3);
+---------------+
| IF(1>2, 2 ,3) |
+---------------+
|             3 |
+---------------+
1 row in set (0.00 sec)

mysql> SELECT IF(1<2, 4 ,5);
+---------------+
| IF(1<2, 4 ,5) |
+---------------+
|             4 |
+---------------+
1 row in set (0.00 sec)

IFNULL
mysql> SELECT IFNULL('A', 'ok');
+-------------------+
| IFNULL('A', 'ok') |
+-------------------+
| A                 |
+-------------------+
1 row in set (0.00 sec)

mysql> SELECT IFNULL(NULL, 'ok');
+--------------------+
| IFNULL(NULL, 'ok') |
+--------------------+
| ok                 |
+--------------------+
1 row in set (0.00 sec)

mysql> SELECT IFNULL('', 'ok');
+------------------+
| IFNULL('', 'ok') |
+------------------+
|                  |
+------------------+
1 row in set (0.00 sec)

# eg: 当score为null时 转换为unknown
mysql> select sid,couerse,score,ifnull(score,'unkuown') from scores;


```



## 字符串函数

###字符串函数介绍

#### ascii码转换 字符长度和拼接

- 字符转换 ACSII(str)

  - 返回str字符串最左边字符的ascii码值，如果是空串则返回0，如果str是null则返回null

- 字符转换 char('64')

  - 将ascii码转换成字符串

- 计算字符长度 Char_length

  - 返回字符串的长度
  - select char_length('wangpenghong');

- 字符串拼接

  - select concat('a', ' ','b')  # 不能出现null，负责结果全部为null
  - select concat_ws('@', 'a', ''b)  # ws指定连接分隔符

  

```sql
mysql> select ascii('a');
+------------+
| ascii('a') |
+------------+
|         97 |
+------------+
1 row in set (0.00 sec)

mysql> select char('97');
+------------+
| char('97') |
+------------+
| a          |
+------------+
1 row in set (0.01 sec)

mysql> select char_length('penghong') as my_long;
+---------+
| my_long |
+---------+
|       8 |
+---------+
1 row in set (0.00 sec)

mysql> select  concat('hello , ', 'world');
+-----------------------------+
| concat('hello , ', 'world') |
+-----------------------------+
| hello , world               |
+-----------------------------+
1 row in set (0.00 sec)
mysql> select * from students2;
+------+-----------+------------+
| sid  | last_name | first_name |
+------+-----------+------------+
|   66 | wang      | penghong   |
+------+-----------+------------+
1 row in set (0.00 sec)

mysql> select concat(last_name,first_name) from students2 as name;
+------------------------------+
| concat(last_name,first_name) |
+------------------------------+
| wangpenghong                 |
+------------------------------+
1 row in set (0.00 sec)

mysql> select concat(last_name,"·",first_name) from students2 as name;
+-----------------------------------+
| concat(last_name,"·",first_name)  |
+-----------------------------------+
| wang·penghong                     |
+-----------------------------------+
1 row in set (0.00 sec)

mysql> select concat('sid: ',sid, " ",last_name,"·",first_name) from students2;
+----------------------------------------------------+
| concat('sid: ',sid, " ",last_name,"·",first_name)  |
+----------------------------------------------------+
| sid: 66 wang·penghong                              |
+----------------------------------------------------+
1 row in set (0.00 sec)

mysql> select concat_ws('@',last_name, first_name) from students2;
+--------------------------------------+
| concat_ws('@',last_name, first_name) |
+--------------------------------------+
| wang@penghong                        |
+--------------------------------------+
1 row in set (0.00 sec)

# 如果指定分割符拼接的字符串有null，则忽略null和null前面的符号
mysql> select concat_ws('@',last_name, first_name, null) from students2;
+--------------------------------------------+
| concat_ws('@',last_name, first_name, null) |
+--------------------------------------------+
| wang@penghong                              |
+--------------------------------------------+
1 row in set (0.00 sec)

```



#### 字符替换 位置查询 字节长度

- insert(str, pos, len,newstr)
  - 将str中从pos位置开始后的len字符串替换成newstr

- instr(str, substr)
  - 查询str字符串中首次出现substr的位置
- left(str, len)
  - 返回字符串str从左边开始的len个长度的字符
- length(str)
  - 返回字符串的byte字节长

- locate(substr, str, pos)
  - 返回str中从pos开始第一次出现substr的位置，没有则返回null
  - 如没有pos，与insert参数位置相反，查询str字符串中首次出现substr的位置



```SQL
# char_length 与 length  字符个数与字节数
mysql> select char_length('鸿飞');
+-----------------------+
| char_length('鸿飞')   |
+-----------------------+
|                     2 |
+-----------------------+
1 row in set (0.01 sec)

mysql> select length('鸿飞');
+------------------+
| length('鸿飞')   |
+------------------+
|                6 |
+------------------+
1 row in set (0.00 sec)
```



#### 大小写转换

- lower(str)
  - 把字符串中字母都变成小写
- upper(str)
  - 把字符串中字母都变成大写



#### 去除空格

- ltrim(str)
  - 将str最左边的空格去掉并返回
- rtrim(str)
  - 将str最左边的空格去掉并返回

- trim(str)
  - 将str两段的空格去掉并返回

```SQL
mysql> select length(ltrim('wph'));
+----------------------+
| length(ltrim('wph')) |
+----------------------+
|                    3 |
+----------------------+
1 row in set (0.00 sec)

mysql> select length(ltrim(' wph'));
+-----------------------+
| length(ltrim(' wph')) |
+-----------------------+
|                     3 |
+-----------------------+
1 row in set (0.00 sec)

mysql> select length(ltrim(' wph '));
+------------------------+
| length(ltrim(' wph ')) |
+------------------------+
|                      4 |
+------------------------+
1 row in set (0.00 sec)

mysql> select length(rtrim(' wph '));
+------------------------+
| length(rtrim(' wph ')) |
+------------------------+
|                      4 |
+------------------------+
1 row in set (0.00 sec)

mysql> select length(trim(' wph '));
+-----------------------+
| length(trim(' wph ')) |
+-----------------------+
|                     3 |
+-----------------------+
1 row in set (0.00 sec)
```



#### 字符串重复、替换、和取出

- repeat(str, count)
  - 将str重复count次并组成字符串返回，如果count<1,则返回空串
- replace(str, from_str, to_str)
  - 将str中匹配的from_str都替换成to_str

- Substr(str, pos, len)
  - 等同于substring
  - 如果没有len，则返回从pos开始的str中的子字符串
  - 如果有len，则从pos位置开始返回str中长度为len的子字符串
  - 如果pos为负值，则从右边开始取值

```SQL
mysql> select substring('abcdefg', 2, 4);
+----------------------------+
| substring('abcdefg', 2, 4) |
+----------------------------+
| bcde                       |
+----------------------------+
1 row in set (0.00 sec)

mysql> select substring('abcdefg', -2, 4);
+-----------------------------+
| substring('abcdefg', -2, 4) |
+-----------------------------+
| fg                          |
+-----------------------------+
1 row in set (0.00 sec)

mysql> select substring('abcdefg', -2);
+--------------------------+
| substring('abcdefg', -2) |
+--------------------------+
| fg                       |
+--------------------------+
1 row in set (0.00 sec)

mysql>
```



## 字符串对比函数



![](https://tva1.sinaimg.cn/large/00831rSTly1gd2v656h51j30jo0bpwiu.jpg)

- like ||  not like
  - 使用"\\"屏蔽“%”和“_”的特殊含义
  - “%“ 任意匹配
  - "_" 匹配一个字符



## 数字函数之算数操作符

- / 和 div
  - / 代表除法
  - div代表整除，只取整数部分
- mod 和 %
  - 用法 mod(x,y)  or   x%y  or  x mod y
  - 取模，取余数

```SQL
mysql> select 7/2;
+--------+
| 7/2    |
+--------+
| 3.5000 |
+--------+
1 row in set (0.00 sec)

mysql> select 7 div 2;
+---------+
| 7 div 2 |
+---------+
|       3 |
+---------+
1 row in set (0.00 sec)

mysql> select 7 mod 2;
+---------+
| 7 mod 2 |
+---------+
|       1 |
+---------+
1 row in set (0.00 sec)

mysql> select 7 % 2;
+-------+
| 7 % 2 |
+-------+
|     1 |
+-------+
1 row in set (0.00 sec)
```



## 数字函数

- ABS(x)
  - 取绝对值
- CEILING(x) ||CEIL(x)
  - 返回大于等于x的最小整数
- FLOOR(x)
  - 返回小于等于x的最大整数

```SQL
mysql> select ceil(1.2);
+-----------+
| ceil(1.2) |
+-----------+
|         2 |
+-----------+
1 row in set (0.00 sec)

mysql> select ceil(-1.2);
+------------+
| ceil(-1.2) |
+------------+
|         -1 |
+------------+
1 row in set (0.00 sec)

mysql> select floor(-1.2);
+-------------+
| floor(-1.2) |
+-------------+
|          -2 |
+-------------+
1 row in set (0.00 sec)

mysql> select floor(1.2);
+------------+
| floor(1.2) |
+------------+
|          1 |
+------------+
1 row in set (0.00 sec)
```

- rand([N])
  - 获取0-1之间随机小数
  - 如果想获取7~12间随机整数可以用` select  floor(7+rand()*5)；`
  - 随机查现有表的一条数据`select * from students order by rand limit 1;`，相当于虚构了rand()字段

```SQL
mysql>  select  floor(7+rand()*5);
+-------------------+
| floor(7+rand()*5) |
+-------------------+
|                10 |
+-------------------+
1 row in set (0.00 sec)

mysql>  select  floor(7+rand()*5);
+-------------------+
| floor(7+rand()*5) |
+-------------------+
|                 9 |
+-------------------+
1 row in set (0.00 sec)
```

- round(x) || round(x,d)
  - 四舍五入为d位小数，d不存在时，默认为0
  - `sentct round(1.58,1) ==> 1.6`
- truncate(x, d)
  - 截断后面的小数，只截断，不进1。
  - `select truncate(1.58,1) ==> 1.5`

## 日期和时间函数

### 日期和时间获取

- curdate(), current_date, current_date()

  ```SQL
  mysql> select curdate(), current_date, current_date();
  +------------+--------------+----------------+
  | curdate()  | current_date | current_date() |
  +------------+--------------+----------------+
  | 2020-03-22 | 2020-03-22   | 2020-03-22     |
  +------------+--------------+----------------+
  1 row in set (0.00 sec)
  ```
  - 

- curtime(), current_time, current_time()

  ```SQL
  mysql> select curtime(), current_time, current_time();
  +-----------+--------------+----------------+
  | curtime() | current_time | current_time() |
  +-----------+--------------+----------------+
  | 18:40:30  | 18:40:30     | 18:40:30       |
  +-----------+--------------+----------------+
  1 row in set (0.00 sec)
  ```

- NOW()

  ```sql
  mysql> SELECT NOW();
  +---------------------+
  | NOW()               |
  +---------------------+
  | 2020-03-22 18:42:12 |
  +---------------------+
  1 row in set (0.00 sec)
  ```

  

- DATE(expr)

  - 从expr中只获取日期，不包含钟点时间

- TIME(expr)

  - 从expr中获取钟点时间，不包含日期

  ```SQL
  mysql> select now(), date(now()),time(now());
  +---------------------+-------------+-------------+
  | now()               | date(now()) | time(now()) |
  +---------------------+-------------+-------------+
  | 2020-03-22 18:45:13 | 2020-03-22  | 18:45:13    |
  +---------------------+-------------+-------------+
  1 row in set (0.00 sec)
  ```

  

###时间差计算

- datediff(expr1, expr2)
  - 获取expr1和expr2的天数差异，忽略时分秒
- timediff(expr1, expr2)
  - 获取expr1和expr2的时间差

```sql
mysql> select datediff('2020-03-22 18:45:13', '2019-03-22 18:45:13');
+--------------------------------------------------------+
| datediff('2020-03-22 18:45:13', '2019-03-22 18:45:13') |
+--------------------------------------------------------+
|                                                    366 |
+--------------------------------------------------------+
1 row in set (0.00 sec)

mysql> select datediff('2020-03-22 18:45:13', '2020-03-22 18:40:13');
+--------------------------------------------------------+
| datediff('2020-03-22 18:45:13', '2020-03-22 18:40:13') |
+--------------------------------------------------------+
|                                                      0 |
+--------------------------------------------------------+
1 row in set (0.00 sec)

mysql> select timediff('2020-03-22 18:45:13', '2020-03-22 18:40:13');
+--------------------------------------------------------+
| timediff('2020-03-22 18:45:13', '2020-03-22 18:40:13') |
+--------------------------------------------------------+
| 00:05:00                                               |
+--------------------------------------------------------+
1 row in set (0.03 sec)

mysql> select timediff('2020-03-22 18:45:13', '2020-03-22 18:40:22');
+--------------------------------------------------------+
| timediff('2020-03-22 18:45:13', '2020-03-22 18:40:22') |
+--------------------------------------------------------+
| 00:04:51                                               |
+--------------------------------------------------------+
1 row in set (0.00 sec)

mysql> select timediff('2020-03-22 18:45:13', '2020-03-21 18:40:22');
+--------------------------------------------------------+
| timediff('2020-03-22 18:45:13', '2020-03-21 18:40:22') |
+--------------------------------------------------------+
| 24:04:51                                               |
+--------------------------------------------------------+
1 row in set (0.00 sec)
```



### 时间增减操作

- data_add(date,INTERVAL  expr unit)

  - 时间加法（加负数可以变成减法，等同data_sub()）

- data_sub(date,INTERVAL expr unit)

  - 时间减法，与data_add用法一致

  ```sql
  mysql> select now(), date_add(now(), interval 1 hour);
  +---------------------+----------------------------------+
  | now()               | date_add(now(), interval 1 hour) |
  +---------------------+----------------------------------+
  | 2020-03-23 00:09:14 | 2020-03-23 01:09:14              |
  +---------------------+----------------------------------+
  1 row in set (0.00 sec)
  
  mysql> select now(), date_add(now(), interval 1 day);
  +---------------------+---------------------------------+
  | now()               | date_add(now(), interval 1 day) |
  +---------------------+---------------------------------+
  | 2020-03-23 00:09:28 | 2020-03-24 00:09:28             |
  +---------------------+---------------------------------+
  1 row in set (0.00 sec)
  
  # 同时添加时分秒
  mysql> select now(), date_add(now(), interval '1:1:1' hour_second);
  +---------------------+-----------------------------------------------+
  | now()               | date_add(now(), interval '1:1:1' hour_second) |
  +---------------------+-----------------------------------------------+
  | 2020-03-23 00:11:07 | 2020-03-23 01:12:08                           |
  +---------------------+-----------------------------------------------+
  1 row in set (0.00 sec) 
  ```



### 时间格式化

- data_format(date, new_format)

  - [时间格式大全](https://www.w3school.com.cn/sql/func_date_format.asp)
  - %Y  四位年份    %y 两位年份
  - %m 月               %M 英文单词月份
  - %d  日             
  - %H  时
  - %i    分
  - %S   秒

  ```SQL
  mysql> select now(),date_format(now(), '%Y%m%d %H%i%S');
  +---------------------+-------------------------------------+
  | now()               | date_format(now(), '%Y%m%d %H%i%S') |
  +---------------------+-------------------------------------+
  | 2020-03-23 00:17:54 | 20200323 001754                     |
  +---------------------+-------------------------------------+
  1 row in set (0.01 sec)
  ```



### Day相关函数

- day(date), Dayofmonth(Date)
  - 返回date中日期是当前月份的第几天
- dayname(date)
  - 返回date中日期是星期几 - 英文
- dayofweek(date)
  - 返回date中日期是星期中第"几"天  (周日为第一天)
- dayofyear(date)
  - 返回date中日期是当前年份的第几天

```SQL
mysql> select now(),day(now());
+---------------------+------------+
| now()               | day(now()) |
+---------------------+------------+
| 2020-03-23 00:30:39 |         23 |
+---------------------+------------+
1 row in set (0.00 sec)

mysql> select now(),dayname(now());
+---------------------+----------------+
| now()               | dayname(now()) |
+---------------------+----------------+
| 2020-03-23 00:30:46 | Monday         |
+---------------------+----------------+
1 row in set (0.00 sec)

mysql> select now(),dayofweek(now());
+---------------------+------------------+
| now()               | dayofweek(now()) |
+---------------------+------------------+
| 2020-03-23 00:30:51 |                2 |
+---------------------+------------------+
1 row in set (0.00 sec)
```



### 获取日期中的部分

- EXTRACT(unit FROM date)

  - unit单元和date_add/date_sub函数中的一样，是获取date日期的unit部分

  ```SQL
  mysql> select now(), extract(year from now());
  +---------------------+--------------------------+
  | now()               | extract(year from now()) |
  +---------------------+--------------------------+
  | 2020-03-23 00:35:18 |                     2020 |
  +---------------------+--------------------------+
  1 row in set (0.00 sec)
  
  mysql> select now(), extract(month from now());
  +---------------------+---------------------------+
  | now()               | extract(month from now()) |
  +---------------------+---------------------------+
  | 2020-03-23 00:35:21 |                         3 |
  +---------------------+---------------------------+
  1 row in set (0.00 sec)
  
  mysql> select now(), extract(day from now());
  +---------------------+-------------------------+
  | now()               | extract(day from now()) |
  +---------------------+-------------------------+
  | 2020-03-23 00:35:23 |                      23 |
  +---------------------+-------------------------+
  1 row in set (0.00 sec)
  ```



### unix-time

- UNIX_TIMESTAMP()   UNIX_TIMESTAMP(date)

  - 如果没有date参数，则返回当前时间到1970-01-01 00:00:00之间的秒数
  - 如果有date参数，则返回date到1970-01-01 00:00:00之间的秒数

  ```SQL
  mysql> select now(), unix_timestamp();
  +---------------------+------------------+
  | now()               | unix_timestamp() |
  +---------------------+------------------+
  | 2020-03-23 00:38:01 |       1584895081 |
  +---------------------+------------------+
  1 row in set (0.00 sec)
  
  mysql> select now(), unix_timestamp('2020-01-01 00:00:01');
  +---------------------+---------------------------------------+
  | now()               | unix_timestamp('2020-01-01 00:00:01') |
  +---------------------+---------------------------------------+
  | 2020-03-23 00:39:01 |                            1577808001 |
  +---------------------+---------------------------------------+
  ```

  

- from_unixtime(date);

  - 从unix时间获取日期格式时间  绝对零时到底是啥？？？

  ```SQL
  mysql> select from_unixtime('0');
  +----------------------------+
  | from_unixtime('0')         |
  +----------------------------+
  | 1970-01-01 08:00:00.000000 |
  +----------------------------+
  1 row in set (0.00 sec)
  ```

  

- last_day(date)

  - 获取当前月份的最后一天

  ```SQL
  mysql> select now(), last_day(now());
  +---------------------+-----------------+
  | now()               | last_day(now()) |
  +---------------------+-----------------+
  | 2020-03-23 00:50:50 | 2020-03-31      |
  +---------------------+-----------------+
  1 row in set (0.00 sec)
  ```
