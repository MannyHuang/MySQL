## sql常见命令
mysql 启动
net start 服务器名

mysql 停止
net stop 服务器名

mysql 自带客户端登陆，只限于root用户
mysql 终端登陆（管理员身份）
mysql -h localhost -P 端口号 -u 用户名 -p用户名 （之后输入密码）

mysql 退出
exit  /  ctrl+c

show database
use 库名 进入对应的库
show tables 查看当前库
show tables from 库名 查看别的库
desc 表名  查看表结构
select version()  查看服务器版本

## 数据定义语言 DDL -- 库和表的管理
### 库的管理
创建、修改（一般不修改）、删除

1.
create database 库名；
create database if not exists 库名；
2.
更改库的字符集
alter database 库名 character set gbk；
3.
drop database if exists 库名；

### 表的管理    ！重要
创建、修改、删除

1. 创建
create table 表名(
  列名 列的类型（长度） 约束，
  列名 列的类型（长度） 约束，
  ...,
  列名 列的类型（长度） 约束，
)

eg:
create table book(
  id INT,
  bName VARCHAR(20),
  price DOUBLE,
  authorId INT,
  publishDate DATETIME
)

2. 修改
- 修改列名
alter table 表名 change column 旧列名 新列名 新类型 （新约束）;

- 修改列的类型或约束
alter table 表名 modify column 列名 新类型 （新约束）;

- 添加新列
alter table 表名 ADD column 列名 类型 （FIRST / AFTER 列名）;

- 删除列
alter table 表名 DROP column 列名

- 修改表名
alter table 表名 RENAME （TO） 新表名

3. 删除
drop table 表名；
drop table if exists 表名；

show tables；

4. 表的复制（可以跨库，from 库名.表名）

- 只复制表结构
create table 表名 like 旧表名；

- 复制表结构+数据
create table 表名
select * from 旧表名；

- 只复制部分数据
create table 表名
select 列名 from 旧表名
where 条件； 

- 只复制某些字段
create table 新表名
select 列名 from 旧表名
where 0；

### 常见数据类型
-数值型
  - 整型 Tinyint、Smallint、Mediumint、Int/integer、Bigint  
    - 1.默认有符号，当约束：unsigned为无符号整型；
    - 2.插入值超出范围，报错：out of range，并且插入临界值
    - 3.如果不设置长度，无符号默认11，有符号默认10。长度代表显示的最大宽度，如果不够会用0在左边填充（约束：zerofill）
  - 小数
    - 定点型：dec(M,D)、decimal(M,D)（默认为（10，0））
    - 浮点型：float(M,D)、double(M,D)，根据插入的数值精度决定精度
    - (M,D)可省略。M：整数位+小数位的最大数 D：保留的小数点位数，如果超过范围，则插入临界值
    - 定点型精度更高，如果要求插入数值的精度较高如货币运算等则考虑使用定点型， 无要求浮点型更好，占用字节更少
- 字符型
  - 较短的文本：char(M)（M可省，默认为1，固定长度的字符，空间比较耗费，效率更高）、varchar(M)（M不可省，可变长度的字符，空间比较节省，效率更低）
  - 较长的文本：text、blob（较长的二进制数据）
  - binary、varbinary，保存较短的二进制
  - enum，枚举型，只能插入设置的值，一次只能选一个
  - set，一次可以选取多个，根据成员个数不同，存储所占字节也不同
- 日期型
  - date
  - datetime，只能反映插入时的当地时区
  - datestamp，支持的时间范围较小，和实际时区有关，更能反映实际的日期，属性受Mysql版本和SQLMode的影响很大
  - time
  - year


### 常见约束
一种限制，用于限制表中的数据，为了保障表中数据的一致性

1. NOT NULL 非空约束， 用于保证该字段的值不为空，eg：姓名、学号等
2. DEFAULT 默认， 保证该字段有默认值，eg：性别默认为男
3. PRIMARY KEY 主键， 保证该字段的值具有唯一性，并且非空，eg：学号、员工编号
4. UNIQUE 唯一， 保证该字段的值唯一，可以为空， eg：座位号
5. CHECK 检查约束， mysql不支持（不报错无效果）eg: 年龄、性别
6. FOREIGN KEY 外键 用于限制两个表的关系，保证该字段的值必须来自于主表的关联列的值。
                   在从表中添加外键约束，用于引用主表中某列的值。 eg：学生表的专业编号，员工表的部门编号


添加约束的时机：
- 创建表时
- 修改表时（有结构未赋值）

约束的添加分类：
- 列级约束
  六大约束语法上都支持，但外键约束没有效果
- 表级约束
  除了非空、默认，其他都支持



#### 创建表时添加约束
1. 添加列级约束

```sql
CREATE TABLE major (
  id INT PRIMARY KEY,
  majorName VARCHAR(20)
);


CREATE TABLE stuinfo (
  id INT PRIMARY KEY,
  stuName VARCHAR(20) NOT NULL,
  gender CHAR(1) CHECK(gender='male' OR gender='female'), ## not support
  seat INT UNIQUE,
  age INT DEFAULT 18,
  majorId INT REFERENCE major(id)   ## not support
);
```



2. 添加表级约束

```sql
CREATE TABLE stuinfo (
  id INT,
  stuName VARCHAR(20),
  gender CHAR(1),
  seat INT,
  age INT,
  majorId INT,

  CONSTRAINT pk PRIMARY KEY(id),
  CONSTRAINT uq UNIQUE(seat),
  CONSTRAINT ck CHECK(gender='male' OR gender='female'),
  CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorid) REFERENCES major(id) 
);
```
constraint （约束名） 约束类型（字段名）   约束名可省略

通用写法：
```sql
CREATE TABLE stuinfo (
  id INT PRIMARY KEY,
  stuName VARCHAR(20) NOT NULL,
  gender CHAR(1),
  seat INT UNIQUE,
  age INT DEFAULT 18,
  CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorid) REFERENCES major(id)
);
```

主键 vs 唯一
- 都能保证唯一性
- 主键不能为空，UNIQUE可以
- 主键最多一列，UNIQUE可以多个
- 都允许组合，但不稳定，不推荐

外键
- 要求在从表设置关联关系
- 从表的外键列的类型和主表的关联列的类型要求一致/兼容，名称无要求
- 主表的关联列必须是一个key（主键或UNIQUE）
- 插入数据时，先插入主表，再插入从表；删除数据时，先删从表，再删主表


#### 修改表时添加约束
1. 添加非空、默认
alter table 表名 modify column 列名 类型 列级约束

2. 主键/UNIQUE既是列级约束又是表级约束，所以添加主键/UNIQUE有两种写法：
- alter table 表名 modify column 列名 类型 PRIMARY KEY / UNIQUE
- alter table 表名 add primary key（列名） / unique（列名）

3. 添加外键
alter table 表名 add foreign key（列名） references 主表名（列名）



#### 修改表是删除约束
1. 添加非空、默认
alter table 表名 modify column 列名 类型 NULL

2. 删除主键
alter table 表名 drop primary key

3. 删除唯一
alter table 表名 drop index 列名

4. 删除外键
alter table 表名 drop foreign key 列名

### 标识列 / 自增长列
不用手动插入值，系统提供默认的序列值
- 标识列不一定和主键搭配，但要求是一个key（unique也可以）
- 一个表中至多有一个标识列
- 标识列只能是数值型
- auto_increment步长可用变量设置，起始值mysql不支持变量设置，可以手动插入


1. 创建表时设置标识列
create table 表名 （
  列名 类型 约束 auto_increment
） 
插入值时该列名值的位置用NULL代替，默认起始值为1

2. 修改表时设置标识列
alter table 表名 modify column 列名 类型 约束 auto_increment

3. 修改表时删除标识列
alter table 表名 modify column 列名 类型 约束；

--------------------------

## 数据操作语言 DML
插入，修改，删除

1. 插入
方式一： 
insert into 表名（列名，...） values （值1， ...）；
1） 插入的值类型要与列的类型一致/兼容
2） 不为null的列必须插入值。可为null的列插入值的方法：
  - 插入列名的时候值用NULL
  - 不写列名，不写值，自动填充null
3） 列的顺序可以调换，但要与值一一对应
4） 可以省略列名，默认所有列，且有顺序

方式二：
insert into 表名
set 列名=值，列名=值，...

对比：
1. 方式一支持多行一起插入，每行用逗号隔开，方式二不行
2. 方式一支持将子查询作为值插入到表中，方式二不支持



2. 修改
- 修改单表记录
update 表名
set 列=新值， 列=新值， ...
where 筛选条件； （不加where整表修改）

- 修改多表记录
sql92语法：
update 表1 别名，表2 别名
set 列=值，...
where 连接条件
and 筛选条件；

sql99语法：
update 表1 别名
连接类型 join 表2 别名
on 连接条件
set 列=值，...
where 筛选条件

3. 删除
方式一： 
delete from 表名 where 筛选条件 （不加筛选则全删）
多表时：
  - sql92
  delete 表1的别名，表2的别名
  from 表1 别名， 表2 别名
  where 连接条件
  and 筛选条件

  - sql99
  delete 表1的别名，表2的别名
  from 表1 别名
  连接类型 join 表2 别名
  on 连接条件
  where 筛选条件


方式二：
truncate table 表名； 删除整张表，不能加where

区别：
1. delete可以加where，truncate不行
2. truncate效率更高
3. 假如要删除的表中有自增长列，用delete删除后再插入数据，自增长列的值从断点开始，用truncate则增1开始
4. truncate删除没有返回值，delete有，希望返回‘x行受影响’则用delete
5. truncate删除不能回滚，delete可以

--------------------------

## 查询
### 函数
- 单行函数
一行产生一个值
  - 字符函数
    - length，中文，如果是utf-8，一个字计3个字符，如果是jbk，一个字计2个字符
    - concat
    - upper、lower
    - substr、substring 截取从指定索引处（指定长度）字符串
    - instr  返回子串第一次出现的索引，如果找不到返回0
    - trim 去掉前后空格； trim（‘子串’ from ‘字符串’）可以去掉字符串前后的子串，中间的保留
    - lpad('str', index, 'substr') 用指定字符左填充指定长度,index指最终的字符数不是长度
    - rpad
    - replace('str', 'substr', 'replacestr') 全部替换
  - 数学函数
    - round(num)， 除去正负号的部分四舍五入为整数，round(num,idx), 保留inx位小数
    - ceil 向上取整，返回>=该参数的最小整数
    - floor 向下取整
    - truncate 截断
    - mod(a,b) 取余 等价于 % ，a为被除数，b为除数，余数正负号取决于被除数的正负。原理：a-a/b*b
  - 日期函数
    - now 返回当前系统日期+时间
    - curdate 返回系统当前日期，无时间
    - curtime 返回当前时间，无日期
    - YEAR、MONTH、MONTHNAME、DAY、HOUR、MINUTE、SECOND
    - str_to_date('date_str','format') 将日期格式的字符转换成指定格式的日期 eg,format='%Y-%c-%d'
    - date_format  将日期转换成字符
    - datediff 
  - 其他函数
    - VERSION，DATABASE，USER
  - 流程控制函数
    - if(exp1,exp2,exp3), 如果exp1成立，返回exp2，否则返回exp3
    - case函数使用一
      case 要判断的字段或表达式
      when 常量1 then 要显示的值1或语句1
      when 常量2 then 要显示的值2或语句2
      ...
      else 要显示的值n或语句n
      end
    - case函数使用二：类似于多重if
      case 
      when 条件1 then 要显示的值1或语句1
      when 条件2 then 要显示的值2或语句2
      ...
      else 要显示的值n或语句n
      end

- 分组/聚合/统计函数
多行产生一个值
  - 简单函数（忽略null；可以和distinct搭配实现去重运算；和分组函数一同查询的字段要求是group by后的字段）
    - sum 处理数值型，字符型不报错但无意义，（null+任何值都为null）
    - avg 处理数值型，字符型不报错但无意义
    - max 处理任意类型
    - min 处理任意类型
    - count 处理任意类型 count(*), count(常量),count(字段)都可以统计行数。
    效率：
      MYISAM存储引擎下，count(*)效率高
      INNODB存储引擎下，count(*)和count(1)差不多，比count(字段)好
  - 分组查询
    - 分组前的筛选：数据源为原始表，用where, 在group by前面，能用分组前筛选优先用分组前
    - 分组后的筛选：数据源为分组后的结果集，用having, 在group by后面，分组函数肯定用having（原始表中没有）
    - mysql分组/筛选语句可以用别名，oracal不行
    - group by 支持多个字段分组，表达式，函数

### 连接查询 / 多表查询
当查询的字段来自多个表时用到

1.笛卡尔乘积现象：表1 m行， 表2 n行，连接查询后m*n行
  - 发生原因：没有有效的连接条件
  - 如何避免：添加有效的连接条件

2.分类
  - 按年代分类
    - sql92标准：仅支持内连接
    - sql99标准 （推荐）：支持内连接、外连接（左外+右外）、交叉连接
  - 按功能分类
    - 内连接
      - 等值连接
      - 非等值连接
      - 自连接
    - 外连接  用于查询一个表中有，另一个表中没有的记录。查询结果显示主表的全部记录，主表中有从表中没有的会显示null。即等于内连接结果+主有从无的记录
      - 左外连接  left左边是主表
      - 右外连接  right右边的是主表 
      - 全外连接  内连接+表1有表2无+表2有表1无  mysql不支持，oracal支持
    - 交叉连接  笛卡尔

#### sql92等值连接
多表等值连接的结果为多表的交集部分
n表连接， 至少要n-1ge连接条件
多表的顺序没有要求
一般需要为表起别名，起了别名后原名失效
可以搭配排序、分组、筛选等子句使用


#### sql92非等值连接
表的连接条件不再是等号，可以是between and
可以搭配排序、分组、筛选等子句使用

#### sql92自连接
同一张表，不同字段之间有对应关系

#### sql99

select 查询列表
from 表1 别名 连接类型
join 表2 别名
on 连接条件
（where 筛选条件
group by 分组
having 筛选条件
order by 排序列表）
连接条件与筛选条件分离，更可读

连接类型：
内连接 - (inner)
左外  - left （outer）
右外  - right （outer）
全外  - full （outer）
交叉  - cross


### 子查询 / 内查询
出现在其他语句中的select语句，放在小括号内
内部嵌套其他select语句的查询，成为外查询/主查询

- 分类
  - 按子查询出现的位置：
    - select后： 仅支持标量子查询
    - from后： 支持表子查询
    - where/having后： 标量子查询/列子查询/行子查询
    - exists后（相关子查询）： 表子查询
  - 按结果集的行列数不同：
    - 标量子查询（结果集只有一行一列）
    - 列子查询（结果集只有一列多行）
    - 行子查询（结果集有一行多列、多行多例）
    - 表子查询（结果集一般多行多例）


#### where / having 后面
特点：
  - 标量子查询：一般搭配单行操作符使用 > < >= <= <>
  - 列子查询： 般搭配多行操作符使用 in/not in、any/some、all
  - 行子查询： 

#### from 后面
将子查询结果充当一张表，可连接另一张表，要求必须起别名

#### exists 后面
SELECT EXISTS （SELECT 列名 from 表名），括号里为布尔类型，true则查询结果为1，false则为0


### 分页查询
当要显示的数据一页显示不全，需要分页提交sql请求

select 查询列表
from 表
（join type join表2
on 连接条件
where 筛选条件
group by 分组字段
having 分组后筛选
order by 排序字段）
limit offset， size；

offset要显示条目的起始索引（从0开始,默认为0），size要显示的条目个数
limit放在最后，执行顺序也是最后

公式：
要显示的页数page，每页的条目数size
select 查询列表
from 表
limit （page-1）*size， size；


### 联合查询
将多条查询语句的结果合并成一个结果

- 语法
查询语句1
union
查询语句2
...
可以多个union

- 应用场景
查询来自多个表，且各表之间没有直接的连接关系，但查询的信息一致

- 特点
1. 多条查询的列数必须一致
2. 多条查询的每一列的类型和顺序一致 （虽然不一致不会报错）
3. union自动去重，如果不想去重，可以用union all


### 事务控制语言  Transaction Control Language
- 事务：一个或一组sql语句组成一个执行单元，这个执行单元要么全部执行，要么全部不执行。
案例： 转账突然终端，一方出账另一方未入账。如果单元中某条sql语句一旦执行失败或产生错误，整个单元将会回滚。所有受到影响的数据将返回到事务开始以前的状态；如果单元中的所有sql语句均执行成功，则事务被顺利执行。

- 存储引擎：mysql中的数据用各种不同的技术存储在文件（或内存）中
可以通过show engines查看mysql支持的存储引擎
mysql用的最多的有：innodb支持事务，myisam、memory不支持事务

- 事务的ACID属性
  1. 原子性（Atomicity）：事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生
  2. 一致性（Consistency）： 事务必须使数据库从一个一致性状态变换到另一个一致性状态
  3. 隔离性（Isolation）： 一个事务的执行不能被其他事务干扰
  4. 持久性（Durability）： 事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来的其他操作和数据库故障不应该对其有任何影响

- 事务的创建
隐式事务：事务没有明显的开启和结束的标记，比如insert、update、delete语句

显式事务：事务有明显的开启和结束的标记。前提：必须设置自动提交功能禁用
步骤：
1. 开启事务
set autocommit = 0；
start transaction； 可选
2. 编写事务中的sql语句（select、insert、update、delete， create、alter、drop属于DDL，无事务之说）
语句1；
语句2；
...
3. 结束事务
commit；（成功执行） / rollback； （执行失败）

- 数据库的隔离级别
多个事务同时运行，当这些事务访问数据库中相同的数据时，如果没采取必要的隔离机制，就会导致各种并发问题：
  - 脏读：T1读取了已经被T2更新但还没有被提交的字段之后，如果T2回滚，T1读取的内容就是临时且无效的
  - 不可重复读：T1读取了一个字段，然后T2更新该字段后，T1再次读取同一字段值就不同了
  - 幻读： T1读取了一个表的一个字段，T2在表中插入了新行，如果T1再次读取同一个表就会多出几行
数据库提供的4种事务隔离级别：
  - read uncommitted 读未提交数据：允许事务读取未被其他事务提交的变更，脏读、不可重读读和脏读会出现
  - read committed 读已提交数据： 只允许事务读取已经被其他事务提交的变更，可避免脏读，不可重读读和脏读会出现
  - repeated read 可重复读： 确保事务可以多次从一个字段中读取相同的值。在这个事务持续期间，禁止其他事务对这个字段进行更新。可避免脏读和不可重复读，幻读会出现
  - serializable 串行化： 确保事务可以从一个表中读取相同的行。在这个事务持续期间，禁止其他事务对该表执行插入、更新、删除。并发问题都可避免，但性能十分低下
Oracal支持：read commited（默认）、serializable
mysql支持4种，默认repeatable read
select @@tx_isolation 查看当前隔离级别
set session transaction isolation level read uncommitted

- delete和truncate在事务使用时的区别
```sql
set autocommit=0;
start transaction;
delete from account;
rollback  # 没有删除掉，成功回滚

set autocommit=0;
start transaction;
truncate table account；
rollback   # 删除掉，回滚失败
```


#### savepoint 
set autocommit = 0；
start transaction；
delete from account where id=2；
savepoint a； 设置保存点
delete from account where id=8；
rollback to a； rollback只和savepoint一起用

select * from account； 2号删掉了，8号没有


### 视图
虚拟表，和普通表一样使用，重复使用
mysql5.1出现的新特性，通过表动态生成的数据，只保存sql逻辑，不保存查询结果
eg：各班抽出同学组成舞蹈班，应付领导不定期巡查

- 视图vs表
  - 关键词不同
  - 视图只保存sql逻辑，没有实际占用物理空间，表保存数据，实际占用物理空间
  - 视图可增删改查，但一般不能增删改

- 创建
create view 视图名
as
查询语句（一般较复杂）；

- 使用，和表的使用方法一样
select * from 视图名
（where 筛选条件）

- 优点
1. 重用sql语句
2. 简化操作，不必知道查询细节
3. 保护数据，提高安全性  （视图与表相分离，只知结果不知原始数据）

- 视图的修改
方式一：
create or replace view 视图名
as
查询语句；

方式二：
alter view 视图名
as
查询语句；

- 删除视图（要求有删除权限）
drop view 视图名1，视图名2，... 可同时删除多个

- 查看视图（结构）
desc 视图名； 和看表的方法一样 
show create view 视图名； 客户端显示不全，不推荐使用

- 更新视图
1. 插入
insert into 视图名 values （）；  插入后原始表同步更新
2. 修改
update 视图名 set 列名=值 where 筛选条件；  原始表同步更新
3. 删除
delete from 视图名 where 筛选条件；  原始表同步更新

简单的更新视图原始表同步更新缺乏安全，因此通常会为视图赋予权限，如只读

a. 视图中具备以下关键字的sql语句：分组函数、distinct、group by、having、uniob或者union all
b.常量视图
c.select中包含子查询
d.连接 （可更新无法插入）
e.from一个不能更新的视图
f.where子句的子查询引用了from子句中的表
则无法广义的更新


### 变量
- 系统变量：系统提供，属于服务器层面
  - 全局变量
  - 会话变量
- 自定义变量
  - 用户变量
  - 局部变量

#### 系统变量
作用域：服务器每次启动将为所有的全局变量赋初始值，针对所有的会话（连接）有效，但不能跨重启（除非修改配置文件）

1. 查看所有系统变量
show variables；默认会话
show global variables；全局
show session variables；会话

2. 查看满足条件的部分系统变量
show global| session variables like ‘%char%’；

3. 查看指定的某个系统变量的值
select @@global| session.系统变量名；

4. 为某个系统变量赋值
方式一：set global|session 系统变量名 = 值；
方式二：set @@global|session.系统变量名 = 值；
 
会话变量作用域：仅针对于当前会话（连接）有效

#### 自定义变量
变量是由用户自定义的
使用步骤：
  - 声明
  - 赋值
  - 使用：查看、比较、运算等

1. 用户变量  作用域：针对于当前会话（连接）有效，等同于会话变量的作用域
应用在任何地方，即begin end内外

使用步骤：
  - 声明并初始化
  方法一： set @用户变量名=值；
  方法二： set @用户变量名:=值；
  方法三： select @用户变量名:=值；

  - 赋值（更新用户变量的值）
  方式一：通过set或select
          set @用户变量名=值；
          set @用户变量名:=值；
          select @用户变量名:=值；
  方式二：通过select into
          select 字段 into @变量名
          from 表；

  - 使用（查看用户变量名的值）
    select @用户变量名；

2. 局部变量  作用域：仅在定义它的begin end中有效，且是begin end第一句
- 声明
declare 变量名 类型
declare 变量名 类型 default 值；

- 赋值
方式一：通过set或select
          set 用户变量名=值；
          set 用户变量名:=值；
          select @用户变量名:=值；

方式二：通过select into
      select 字段 into 变量名
      from 表；

- 使用
select 局部变量名

用户变量 vs 局部变量
- 用户变量针对当前会话，可在会话中任何地方定义和使用，声明必须加@，不用限定类型
- 局部变量只能在begin end中，一般不用加@，需要限定类型

### 存储过程和函数
好处：提高代码重用性、简化操作

#### 存储过程
存储过程：一组预先编译好的sql语句的集合，理解成批处理语句
减少了编译次数并且减少了和数据库服务器的连接次数，提高了效率 

- 创建
create procedure 存储过程名（参数列表） 
begin
  存储过程体（一组合法的sql语句）
end

注意：
1. 参数列表包含三部分：参数模式、参数名、参数类型
eg： in stuname varchar（20）
参数模式：
  - in：该参数可作为输入，即需要调用方传入值
  - out：该参数可作为输出，即可以作为返回值
  - inout：可以作为输入、输出，即需要传入值，也可返回值

存储过程体不能被修改

2. 如果存储过程仅有一句话，begin end可以省略
存储过程体中的每一条sql语句的结尾要求必须加分号。
存储过程的结尾可以用delimiter重新设置：
delimiter 结束标记
之后的；都需要用这个结束标记替代
 
- 调用
call 存储过程名（实参列表）
1. 空参列表 
2. 创建带in模式参数的存储过程
3. 创建带out模式参数的存储过程
4. 创建带inout模式参数的存储过程

- 删除
drop procedure 存储过程名

- 查看存储过程的信息
show create procedure 存错过程名

#### 函数
存储过程可以有0个返回或多个返回，适合做批量插入、批量更新
函数只有一个，适合处理数据后返回一个结果

- 创建
create function 函数名（参数列表）returns 返回类型
begin
  函数体
end

注意：
1. 参数列表包含：参数名，参数类型
2. 函数体：肯定有return语句，如果没有会报错
如果return语句没有放在函数体的最后也不报错但不建议
3. 函数体只有一句话则可省略begin end
4. 使用delimiter语句设置结束标记

- 调用
select 函数名（参数列表）
1. 无参有返回
2. 有参有返回

- 查看
show create function 函数名

- 删除
drop function 函数名

### 流程控制结构
- 顺序结构： 程序从上往下依次执行
- 分支结构： 程序从两条或多条路径中选择一条执行
- 循环结构： 程序在满足一定条件的基础上，重复执行一段代码

#### 分支结构
1. if函数
功能：实现简单的双分支
if(exp1，exp2，exp3)
执行顺序：如果exp1成立，则if函数返回exp2的值，否则返回exp3的值
应用：任何地方

2. case结构
- 情况1:类似于java中的switch，一般用于等值判断
case 变量|表达式|字段
when 判断值1 when 返回值1|语句1；
when 判断值2 when 返回值2|语句2；
...
else 返回值n|语句n；
end case；

- 情况2:类似于java中的多重if语句，一般用于区间判断
case 
when 判断条件1 when 返回值1|语句1；
when 判断条件2 when 返回值2|语句2；
...
else 返回值|语句n；
end case；

特点：
  1. 可以作为表达式，嵌套在其他语句中使用，可以放在任何地方，begin end内外；可以作为独立的语句使用，只能放在begin end中
  2. 如果when中的值满足或条件成立，则执行对应的then语句，并且结束case；如果都不满足，则执行else中的语句
  3. else可以省略，如果省略了且when条件都不满足，则返回null

3. if结构
实现多重分支
if 条件1 then 语句1；
elseif 条件2 then 语句2；
...
else 语句n；
end if；
只能应用在begin end中

#### 循环结构
分类：while、loop、repeat
循环控制：
iterate类似于continue，结束本次循环，继续下一次
leave类似于break，结束当前所在循环

1. while
（标签：）while 循环条件 do
  循环体；
end while （标签）；
标签为循环控制，加上则表示有循环控制
先判断后执行，可能执行0次

2. loop
（标签：）loop  
  循环体；
end loop （标签）；
可以用来模拟简单的死循环，没有条件的死循环


3. repeat
（标签：）repeat
  循环体；
until 结束循环的条件
end repeat （标签）；
先执行后判断，至少执行一次