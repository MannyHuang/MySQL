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
5. CHECK 检查约束， mysql不支持（不报错无效果）

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






