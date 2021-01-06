# **第1章** ***\*测试\****

## **1.1** ***\*概述\****

推荐书

索引Relational Database Index Design and the Optimizers Tapio Lahdenmaki Mike Leach

### **1.1.1** ***\*检查步骤\****

1 查询慢日志，定位慢sql 

2 对慢sql执行show profile，分析整个语句各步骤的时间

3 对慢sql执行explain, 分析执行计划各步骤的时间，结合2定位sql执行慢的原因

## **1.2** ***\*Show profile\****

### **1.2.1** ***\*概述\****

分析单条语句的时间，可以查询information_schema.PROFILING进一步查询

### **1.2.2** ***\*使用\****

show variables like 'profiling%'; 显示开启状态

set profiling=on; 开启profiling

show profiles; 查询最近所有sql语句

show profile for query XX; 具体查询一条语句，按照查询的具体步骤显示时间，但是不能排序

//更加细致的内容,查询information_schema.PROFILING，可以排序

set @query_id = xx;

SELECT STATE

  ,sum(duration) AS total_r

  ,round(100 * sum(duration) / (

​      SELECT sum(duration)

​      FROM information_schema.PROFILING

​      WHERE query_id = @query_id

​      ), 2) AS pct_r

  ,count(*) AS calls

  ,sum(duration) / count(*) AS "R/Call"

FROM information_schema.profiling

WHERE query_id = @query_id

GROUP BY STATE

ORDER BY total_r DESC;

## **1.3** ***\*explain\****

expain出来的信息有10列，分别是id、select_type、table、type、possible_keys、key、key_len、ref、rows、Extra

 

### **1.3.1** ***\*概要描述：\****

id:选择标识符

select_type:表示查询的类型。

table:输出结果集的表

partitions:匹配的分区

type:表示表的连接类型

possible_keys:表示查询时，可能使用的索引

key:表示实际使用的索引

key_len:索引字段的长度

ref:列与索引的比较

rows:扫描出的行数(估算的行数)

filtered:按表条件过滤的行百分比

Extra:执行情况的描述和说明

 

### **1.3.2** ***\*具体描述\****

#### **1.3.2.1** ***\*一、 id\****

 

SELECT识别符。这是SELECT的查询序列号

 

我的理解是SQL执行的顺序的标识，SQL从大到小的执行

 

\1. id相同时，执行顺序由上至下

 

\2. 如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

 

\3. id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行

 

EXPLAIN SELECT DISTINCT

​	a.id id,

​	a.parent_id parentId,

​	a.image_url imageUrl,

​	a.NAME NAME,

​	a.menu_type menuType,

​	a.menu_path menuPath,

​	a.component_path componentPath,

​	a.default_path defaultPath,

​	a.auth_mark authMark,

​	a.auth_strategy authStrategy,

​	a.menu_event menuEvent,

​	a.sort sort,

​	a.route_menu routeMenu,

​	a.route_hide routeHide,

​	a.route_cache routeCache,

​	a.route_aggregate routeAggregate,

​	a.hpIcon_name hpIconName,

​	a.hpIcon_url hpIconUrl,

​	b.id appId,

​	b.NAME appName 

FROM

​	purview_tbl_base_menu a

​	LEFT JOIN (select * from purview_tbl_base_app where use_flag = 1) b ON a.app_id = b.id

​	LEFT JOIN purview_tbl_base_role_menu c ON c.menu_id = a.id

​	LEFT JOIN (select * from purview_tbl_base_role where use_flag = 1) d ON c.role_id = d.id

​	LEFT JOIN purview_tbl_base_user_role e ON e.role_id = d.id 

WHERE

​	 e.user_id = '8a5a7c5d650c3123097b6fd27e0a54e4' 

ORDER BY

a. sort DESC;

![img](file:///C:\Users\gjyang\AppData\Local\Temp\ksohtml26984\wps1.jpg) 

如上 先执行两个子查询 ，即id为3 和id为2的执行计划，再执行id为1的语句，从上往下；

#### **1.3.2.2** ***\*二、select_type\****

 

   示查询中每个select子句的类型

 

(1) SIMPLE(简单SELECT，不使用UNION或子查询等)

 

(2) PRIMARY(子查询中最外层查询，查询中若包含任何复杂的子部分，最外层的select被标记为PRIMARY)

 

(3) UNION(UNION中的第二个或后面的SELECT语句)

 

(4) DEPENDENT UNION(UNION中的第二个或后面的SELECT语句，取决于外面的查询)

 

(5) UNION RESULT(UNION的结果，union语句中第二个select开始后面所有select)

 

(6) SUBQUERY(子查询中的第一个SELECT，结果不依赖于外部查询)

 

(7) DEPENDENT SUBQUERY(子查询中的第一个SELECT，依赖于外部查询)

 

(8) DERIVED(派生表的SELECT, FROM子句的子查询)

 

(9) UNCACHEABLE SUBQUERY(一个子查询的结果不能被缓存，必须重新评估外链接的第一行)

 

 

 

#### **1.3.2.3** ***\*三、table\****

 

显示这一步所访问数据库中表名称（显示这一行的数据是关于哪张表的），有时不是真实的表名字，可能是简称，例如上面的e，d，也可能是第几步执行的结果的简称

 

 

 

#### **1.3.2.4** ***\*四、type\****

 

对表访问方式，表示MySQL在表中找到所需行的方式，又称“访问类型”。

 

常用的类型有： ALL、index、range、 ref、eq_ref、const、system、NULL（从左到右，性能从差到好）

 

ALL：Full Table Scan， MySQL将遍历全表以找到匹配的行

 

index: Full Index Scan，index与ALL区别为index类型只遍历索引树

 

range:只检索给定范围的行，使用一个索引来选择行

 

ref: 表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值

 

eq_ref: 类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件

 

const、system: 当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量，system是const类型的特例，当查询的表只有一行的情况下，使用system

 

NULL: MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。

 

 

 

#### **1.3.2.5** ***\*五、possible_keys\****

 

指出MySQL能使用哪个索引在表中找到记录，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用（该查询可以利用的索引，如果没有任何索引显示 null）

 

该列完全独立于EXPLAIN输出所示的表的次序。这意味着在possible_keys中的某些键实际上不能按生成的表次序使用。

如果该列是NULL，则没有相关的索引。在这种情况下，可以通过检查WHERE子句看是否它引用某些列或适合索引的列来提高你的查询性能。如果是这样，创造一个适当的索引并且再次用EXPLAIN检查查询

 

 

 

#### **1.3.2.6** ***\*六、Key\****

 

key列显示MySQL实际决定使用的键（索引），必然包含在possible_keys中

 

如果没有选择索引，键是NULL。要想强制MySQL使用或忽视possible_keys列中的索引，在查询中使用FORCE INDEX、USE INDEX或者IGNORE INDEX。

 

 

 

#### **1.3.2.7** ***\*七、key_len\****

 

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度（key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的）

 

不损失精确性的情况下，长度越短越好 

 

 

 

#### **1.3.2.8** ***\*八、ref\****

 

列与索引的比较，表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值

 

 

 

#### **1.3.2.9** ***\*九、rows\****

 

 估算出结果集行数，表示MySQL根据表统计信息及索引选用情况，估算的找到所需的记录所需要读取的行数

 

 

 

#### **1.3.2.10** ***\*十、Extra\****

 

该列包含MySQL解决查询的详细信息,有以下几种情况：

 

Using where:不用读取表中所有信息，仅通过索引就可以获取所需数据，这发生在对表的全部的请求列都是同一个索引的部分的时候，表示mysql服务器将在存储引擎检索行后再进行过滤

 

Using temporary：表示MySQL需要使用临时表来存储结果集，常见于排序和分组查询，常见 group by ; order by

 

Using filesort：当Query中包含 order by 操作，而且无法利用索引完成的排序操作称为“文件排序”

 

-- 测试Extra的filesort

explain select * from emp order by name;

Using join buffer：改值强调了在获取连接条件时没有使用索引，并且需要连接缓冲区来存储中间结果。如果出现了这个值，那应该注意，根据查询的具体情况可能需要添加索引来改进能。

 

Impossible where：这个值强调了where语句会导致没有符合条件的行（通过收集统计信息不可能存在结果）。

 

Select tables optimized away：这个值意味着仅通过使用索引，优化器可能仅从聚合函数结果中返回一行

 

No tables used：Query语句中使用from dual 或不含任何from子句

 

-- explain select now() from dual;

 

# **第2章** ***\*具体措施\****

## **2.1** ***\*物理阶段优化\****

### **2.1.1** ***\*字段优化\****

\1. 使用具体字段比字符串好

\2. 小字段比大字段好

### **2.1.2** ***\*范式、反范式\****

\1. 三大范式

(1) 第一范式：每个属性都不可再分 （如年月日）

(2) 第二范式：不存在非主属性对码有部分函数依赖

(3) 第三范式：不存在非主属性对码的传递函数依赖

(4) Bncf：存在着主属性对于码的部分函数依赖与传递函数依赖

\2. 反范式

(1) 建立冗余字段

(2) 建立缓存表（若干表拼凑的冗余表），建立汇总表

### **2.1.3** ***\*索引\****

#### **2.1.3.1** ***\*索引分类\****

(1) 按结构分

① Btree索引

1) 定义

2) 使用

\1. 如果不是按照索引的最左列开始查找，则无法使用索引

\2. 不能跳过索引中的列

\3. 如果查询中有某个列的范围查询，则其右边所有列都无法使用索引优化查找

3) 原因

② Hash索引

③ Rtree索引

④ 全文索引

(2) 聚簇索引与稀疏索引（二级索引）

二级索引保存一级索引的关键字而不是指针

(3) 聚集索引与非聚集索引

#### **2.1.3.2** ***\*索引优点\****

索引减少了服务器需要扫描的数据量。

索引可以帮助服务器避免排序和临时表。

索引可以将随机I/O变为顺序I/O。

#### **2.1.3.3** ***\*缺点\****

1.  btree：如果不是从索引的最左列开始查找，则无法使用索引
2. btree:  不能跳过索引中的列
3. btree：如果某个列使用范围查询，则右边的列都无法使用索引

#### **2.1.3.4** ***\*高效率索引策略\****

##### **2.1.3.4.1** ***\*独立的列\****

##### **2.1.3.4.2** ***\*平衡索引前缀和索引选择性\****

##### **2.1.3.4.3** ***\*多个索引不如合并成一个大的索引并优化顺序\****

##### **2.1.3.4.4** ***\*覆盖索引\****

###### **2.1.3.4.4.1** ***\*定义\****

查询字段包括在索引中

查询字段

用法

原因

适用场景

##### **2.1.3.4.5** ***\*索引扫描做排序\****

##### **2.1.3.4.6** ***\*前缀压缩索引\****

#### **2.1.3.5** ***\*索引案例\****

##### **2.1.3.5.1** ***\*多个过滤条件\****

\1. 使用频率最高的字段作为索引前缀，如果与选择性低的作为前缀这条冲突，考虑用in规避

##### **2.1.3.5.2** ***\*避免多个范围条件\****

\1. 用in规避范围查询字段右边无法在使用索引的问题

 

##### **2.1.3.5.3** ***\*延迟关联\****

## **2.2** ***\*查询优化\****

### 6.1 查询为什么会慢

#### 查询不需要的数据

#### 扫描额外的记录



 

 

 

 

 

 

 

 

 