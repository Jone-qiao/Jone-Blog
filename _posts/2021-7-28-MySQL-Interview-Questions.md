---
layout: post
title: MySQL 面试题
---

## 什么是 SQL？

- 结构化查询语言（Structured Query Language），简称 SQL，是一种数据库查询语言。

## 什么是 MySQL?

- 关系型数据库管理系统

## 数据库三大范式是什么？

- 第一范式：每个列都不能再拆分；
- 第二范式：在第一范式的基础上，非主键列完全依赖于主键，而不能是依赖于主键的一部分；
- 第三范式：在第二范式的基础上，非主键列只依赖于主键，不依赖于其他非主键；

## MySQL 中有关权限的表都有哪些？

- **user 权限表**：记录允许连接到服务器的用户账号信息，里面的权限是全局级的；

  ```mysql
  create user 'ua'@'%' identified by 'passwd'; # 创建 user
  drop user 'ua'@'%';   # 删除 user
  ```

- **db 权限表**：记录各个账号在各个数据库的操作权限；

- **tables_priv 权限表**：记录数据表级的操作权限；

  ```mysql
  grant all privileges on <database>.<table> to 'ua'@'%' with grant option;
  revoke all privileges on <database>.<table> from 'ua'@'%';
  ```

- **columns_priv 权限表**：记录数据列（字段）级的操作权限；

  ```mysql
  grant select(<column_1>, <column_2>), update(<column_1>) on <database>.<table> to 'ua'@'%';
  # column_1 可以被 select 和 update；
  # column_2 只可以被 select；
  ```

- **host 权限表**：（已废弃）

## MySQL 的 binlog 有几种录入格式？分别有什么区别？

```shell
# mysql.conf
binlog-format = row/statement/mixed
```

- **row**：不记录 SQL 语句上下文相关信息，仅保存哪一条记录被修改。非常清楚的记录下每一行数据修改的细节，但是由于很多操作，会导致大量行的改动（比如 alter table），因此这种模式的文件保存的信息太多，**日志量太大**。
- **statement**【陈述，报告】：每一条会**修改数据的 SQL 语句** 都会记录在 Master Node 的 binlog 中。不需要记录每一行的变化，减少了 binlog 日志量，节约了 IO，提高性能。由于 SQL 的执行是有上下文的，因此在保存的时候需要保存相关的信息，同时还有一些使用了函数之类的语句无法被记录复制。
- **mixed**【混合】：从 5.1.8 版本开始，一种折中的方案，普通操作使用 statement 记录，当无法使用 statement 的时候使用 row。

## MySQL 有哪些数据类型？

### 数值类型

### 字符串类型

### 日期和时间类型

### MySQL 常用的存储引擎有哪些？

- Innodb
- MyISAM
- Memory；

## Innodb 和 MyISAM 的区别

- InnoDB 索引是**聚簇索引**，MyISAM 索引是**非聚簇索引**。

- InnoDB 的主键索引的叶子节点存储着行数据，因此主键索引非常高效。
- MyISAM 索引的叶子节点存储的是行数据地址，需要再寻址一次才能得到数据。
- InnoDB 非主键索引的叶子节点存储的是主键和其他带索引的列数据，因此查询时做到覆盖索引会非常高效。

## 索引有哪些优缺点？

### 优点

- 可以大大**加快数据的检索速度**，这也是创建索引的最主要的原因。
- 通过使用索引，可以在查询的过程中，使用优化隐藏器，提高系统的性能。

### 缺点

- 时间方面：创建索引和维护索引要**耗费时间**，具体地，当对表中的数据进行增加、删除和修改的时候，索引也要动态的维护，会降低增/改/删的执行效率；
- 空间方面：索引需要**占物理空间**。

## 怎么创建索引？

- 第一种方式：在执行 CREATE TABLE 时创建索引；

- 第二种方式：使用 ALTER TABLE 命令去增加索引；

  ```mysql
  ALTER TABLE table_name ADD [INDEX|UNIQUE|FULLTEXT] index_name(column_name);
  ```

- 第三种方式：使用 CREATE INDEX 命令创建

  ```mysql
  CREATE [INDEX|UNIQUE|FULLTEXT] INDEX index_name ON table_name (column_name)
  ```

## 简述有哪些索引和作用

- **唯一索引（UNIQUE）**：不允许有俩行具有相同的值；
- **普通索引（INDEX）**；
- **主键索引（PRIMARY  KEY）**：为了保持数据库表与表之间的关系；
- **聚集索引**：表中行的物理顺序与键值的逻辑（索引）顺序相同。
- **非聚集索引**：聚集索引和非聚集索引的根本区别是表记录的排列顺序和与索引的排列顺序是否一致；

- **联合索引**：在创建索引时，并不是只能对一列进行创建索引，可以与主键一样，讲多个组合为索引；

- **全文索引（FULLTEXT）**：全文索引为在字符串数据中进行复杂的词搜索提供有效支持；

## 举例说明聚集索引和非聚集索引的区别

- 比如字典中，用‘拼音’查汉字，就是聚集索引。因为正文中字都是按照拼音排序的。
- 而用‘偏旁部首’查汉字，就是非聚集索引，因为正文中的字并不是按照偏旁部首排序的，我们通过检字表得到正文中的字在索引中的映射，然后通过映射找到所需要的字。

## 主键索引与唯一索引的区别

1. 主键是一种**约束**，唯一索引是一种**索引**，两者在本质上是不同的。
2. 主键创建后一定包含一个唯一性索引，唯一性索引并不一定就是主键。
3. 唯一性索引列允许空值，而主键列不允许为空值。
4. 一个表最多只能创建一个主键，但可以创建多个唯一索引。
5. 主键更适合那些不容易更改的唯一标识，如自动递增列、身份证号等。

## 什么是聚簇索引？何时使用聚簇索引与非聚簇索引

- 聚簇索引：将数据存储与索引放到了一块，找到索引也就找到了数据；
- 非聚簇索引：将数据存储于索引分开结构，索引结构的叶子节点指向了数据的对应行；

## B 树和 B+ 树的区别

- 在 B 树中，你可以将键和值存放在内部节点和叶子节点；但在 B+ 树中，内部节点都是键，没有值，叶子节点同时存放键和值。
- B+ 树的叶子节点有一条链相连，而 B 树的叶子节点各自独立。

## 数据库为什么使用B+树而不是B树

- B 树只适合**随机检索**，而 B+ 树同时支持**随机检索和顺序检索**；
- B+ 树空间利用率更高，可**减少 I/O 次数**，磁盘读写代价更低；
- B+ 树的查询效率更加稳定；
- B- 树在提高了磁盘 IO 性能的同时并没有解决元素遍历的效率低下的问题。B+ 树的叶子节点使用指针顺序连接在一起，只要遍历叶子节点就可以实现整棵树的遍历。而且在数据库中基于范围的查询是非常频繁的，而 B树不支持这样的操作。
- 增删文件（节点）时，效率更高。因为 B+ 树的叶子节点包含所有关键字，并以有序的链表结构存储，这样可很好提高增删效率。

## 什么是聚簇索引？何时使用聚簇索引与非聚簇索引

- 聚簇索引：**将数据存储与索引放到了一块，找到索引也就找到了数据**；
- 非聚簇索引：将数据存储于索引分开结构，索引结构的叶子节点指向了数据的对应行，myisam 通过key_buffer 把索引先缓存到内存中，当需要访问数据时（通过索引访问数据），在内存中直接搜索索引，然后通过索引找到磁盘相应数据，这也就是为什么索引不在 key buffer 命中时，速度慢的原因；

## 索引设计的原则？

1. 适合索引的列是出现在 where 子语句中的列，或者连接子句中指定的列；
2. 基数较小的列，索引效果较差，没有必要在此列上创建索引；
3. 使用短索引，如果对长字符串列进行索引，应该指定一个前缀长度，这样能够节省大量索引空间；
4. 不要过度索引。

## 创建索引的原则

1. 最左前缀匹配原则，组合索引非常重要的原则，MySQL 会一直向右匹配直到遇到范围查询（>、<、between、like）就停止匹配，比如 `a = 1 and b = 2 and c > 3 and d = 4` 如果建立 (a,b,c,d) 顺序的索引，d 是用不到索引的，如果建立 (a,b,d,c) 的索引则都可以用到，a,b,d 的顺序可以任意调整。
2. 较频繁作为查询条件的字段才去创建索引；
3. 更新频繁字段不适合创建索引；
4. 若是不能有效区分数据的列不适合做索引列（如性别，男女未知，最多也就三种，区分度实在太低）；
5. 尽量的扩展索引，不要新建索引。比如表中已经有 a 的索引，现在要加（a, b）的索引，那么只需要修改原来的索引即可。
6. 定义有外键的数据列一定要建立索引；
7. 对于那些查询中很少涉及的列，重复值比较多的列不要建立索引；
8. 对于定义为 text、image 和 bit 的数据类型的列不要建立索引。

## 如何删除索引

```mysql
ALTER TABLE table_name DROP KEY index_name;
```

## Hash索引的劣势

1. Hash 索引进行等值查询比较快，但是却无法进行范围查询；
2. Hash 索引不支持使用索引进行排序；
3. Hash 索引不支持模糊查询以及多列索引的最左前缀匹配；
4. Hash 索引任何时候都避免不了回表查询数据；
5. Hash 碰撞；

## 联合索引是什么？

- MySQL 可以使用多个字段同时建立一个索引，叫做联合索引。
- 在联合索引中，如果想要命中索引，需要按照建立索引时的字段顺序挨个使用，否则无法命中索引。

## 什么是脏读？幻读？不可重复读？

- 脏读：事务 A 已更新字段 a （从 1 更新为 2），事务 B 此时读取了字段 a（2），由于某些原因事务 A rollback（a=1），则事务 B 读取的数据就会是不正确的；
- 幻读：查询（select）某记录是否存在，不存在，准备插入此记录，但执行 insert 时发现此记录已存在，无法插入，此时就发生了幻读。
- 不可重复读：在一个事务的两次查询之中数据不一致，这可能是两次查询过程中间插入了一个事务更新的原有的数据。

## 事务的隔离级别

- 读未提交；
- 读已提交；
- 可重复读；
- 可串行化；

## MySQL 的锁

- 当数据库有**并发事务**的时候，可能会产生数据的不一致，这时候需要一些机制来保证访问的**次序**，锁机制就是这样一个机制；

## 从锁的类别上来讲 MySQL 都有哪些锁？

- 共享锁（读锁）
- 排它锁（写锁）

## InnoDB 存储引擎的锁的算法有哪几种？

- Record lock：单个行记录上的锁；
- Gap lock：间隙锁，锁定一个范围，不包括记录本身；
- Next-key lock：Record lock 和 Gap lock 锁定一个范围，包含记录本身；

## 数据库的乐观锁和悲观锁是什么？怎么实现的？

### 乐观锁

- 假设不会发生并发冲突，只在提交操作时检查是否违反数据完整性。在修改数据的时候把事务锁起来，通过 **version** 的方式来进行锁定。
- 实现方式：一般会使用**版本号机制**或 **CAS** 算法实现。

```mysql
update table set x=x+1, version=version+1 where id=#{id} and version=#{version};
```

### 悲观锁

- 假定会发生并发冲突，屏蔽一切可能违反数据完整性的操作。在查询完数据的时候就把事务锁起来，直到提交事务。
- 实现方式：使用数据库中的锁机制；

```mysql
select status from t_goods where id=1 for update;
```

## 什么是视图？为什么要使用视图？

- 为了提高复杂 SQL 语句的**复用性**和**表操作的安全性**，MySQL 数据库管理系统提供了视图特性。
- 所谓视图，本质上是一种**虚拟表**，在物理上是不存在的，其内容与真实的表相似，包含一系列带有名称的列和行数据。但是，视图并不在数据库中以储存的数据值形式存在。行和列数据来自定义视图的查询所引用基本表，并且在具体引用视图时动态生成。

```mysql
CREATE VIEW view_name AS
SELECT column1, column2
FROM table
WHERE condition;
```

```mysql
SELECT column1 FROM view_name;
```

## 什么是存储过程？

- 存储过程是一个预编译的 SQL 语句，优点是允许模块化的设计，就是说只需要创建一次，以后在该程序中就可以调用多次。如果某次操作需要执行多次 SQL，使用存储过程比单纯 SQL 语句执行要快。

```mysql
DELIMITER //
CREATE PROCEDURE `add_num`(IN n INT)
BEGIN
       DECLARE i INT;
       DECLARE sum INT;

       SET i = 1;
       SET sum = 0;
       WHILE i <= n DO
              SET sum = sum + i;
              SET i = i +1;
       END WHILE;
       SELECT sum;
END //
DELIMITER ;
```

```mysql
CALL add_num(50);
```

## 什么是触发器？

- 触发器是用户定义在关系表上的一类由事件驱动的特殊的存储过程。触发器是指一段代码，当触发某个事件时，自动执行这些代码。

## MySQL 中都有哪些触发器？

- Before Insert
- After Insert
- Before Update
- After Update
- Before Delete
- After Delete

## SQL 语句主要分为哪几类？

### 数据定义语言 DDL

- CREATE，DROP，ALTER

### 数据查询语言 DQL

- SELECT

### 数据操作语言 DML

- INSERT，UPDATE，DELETE

### 数据控制功能 DCL

- GRANT，REVOKE，COMMIT，ROLLBACK

## SQL语句的语法顺序：

1. SELECT
2. FROM
3. JOIN
4. ON
5. WHERE
6. GROUP BY
7. HAVING
8. UNION
9. ORDER BY
10. LIMIT

## 超键、候选键、主键、外键分别是什么？

- 超键：在关系中能**唯一标识**元组的属性集称为关系模式的超键。
  - 一个属性可以为作为一个超键，多个属性组合在一起也可以作为一个超键。
  - 超键包含候选键和主键。
- 候选键：是最小超键，即没有冗余元素的超键。
- 主键：数据库表中对储存数据对象予以唯一和完整标识的数据列或属性的组合。
  - 一个数据列只能有一个主键，且主键的取值不能缺失，即不能为空值（Null）。
- 外键：在一个表中存在的另一个表的主键称此表的外键。

## SQL 约束有哪些？

- NOT NULL;
- UNIQUE;
- PRIMARY KEY
- FOREIGH KEY
- CHECK

## 关联查询

- 交叉连接（CROSS JOIN）
- 内连接（INNER JOIN）
- 外连接（LEFT JOIN/RIGHT JOIN）
- 联合查询（UNION 与 UNION ALL）
- 全连接（FULL JOIN）

## 什么是子查询

1. 条件：一条 SQL 语句的查询结果做为另一条查询语句的条件或查询结果
2. 嵌套：多条 SQL 语句嵌套使用，内部的 SQL 查询语句称为子查询。

```mysql
SELECT column_1 FROM table_1
WHERE column_2
IN (SELECT column_2 FROM table_2 WHERE column_3='**');
```

## MySQL 中 in 和 exists 区别

- in 语句是把外表和内表做 HASH 连接，而 exists 语句是对外表做 loop 循环，每次 loop 循环在对内表进行查询；
- 如何查询的两个表大小相当，那么用 in 和 exists 差别不大。
- 如果两个表中一个较小，一个是大表，则子查询表大的用 exists，子查询表小的用 in。

- not in 和 not exists：如果查询语句使用了 not in，那么内外表都进行**全表扫描**，没有用到索引；而 not extsts 的子查询依然能用到表上的索引。所以无论那个表大，用 not exists 都比 not in 要快。

## varchar与char的区别

### **char **

- char表示**定长**字符串，长度是固定的；
- 如果插入数据的长度小于 char 的固定长度时，则用空格填充；
- 因为长度固定，所以存取速度要比 varchar 快很多，甚至能快 50%，但正因为其长度固定，所以会占据多余的空间，是空间换时间的做法；
- 对于 char 来说，最多能存放的字符个数为 255，和编码无关

### varchar

- varchar 表示**可变长**字符串，长度是可变的；
- 插入的数据是多长，就按照多长来存储；
- varchar 在存取方面与 char 相反，它存取慢，因为长度不固定，但正因如此，不占据多余的空间，是时间换空间的做法；
- 对于varchar来说，最多能存放的字符个数为 65532；

## FLOAT 和 DOUBLE 的区别是什么？

- FLOAT 类型数据可以存储至多 8 位十进制数，并在内存中占 4 字节。
- DOUBLE 类型数据可以存储至多 18 位十进制数，并在内存中占 8 字节。

## SQL 的执行顺序

1. FROM：将数据从硬盘加载到数据缓冲区，方便对接下来的数据进行操作。
2. WHERE：从基表或视图中选择满足条件的元组。（不能使用聚合函数）
3. JOIN（如 right left 右连接-------从右边表中读取某个元组，并且找到该元组在左边表中对应的元组或元组集）
4. ON：join on 实现多表连接查询，推荐该种方式进行多表查询，不使用子查询。
5. GROUP BY：分组，一般和聚合函数一起使用。
6. HAVING：在元组的基础上进行筛选，选出符合条件的元组。（一般与GROUP BY进行连用）
7. SELECT：查询到得所有元组需要罗列的哪些列。
8. DISTINCT：去重的功能。
9. UNION：将多个查询结果合并（默认去掉重复的记录）。
10. ORDER BY：进行相应的排序。
11. LIMIT 1：显示输出一条数据记录（元组）

## 字段为什么要求定义为 not null？

- null 值会占用**更多的字节**，且会在程序中造成很多与预期不符的情况。

## PHP 使用 MySQL

### PDO

```php
# 连接
try {
  $dbh = new PDO('mysql:host=127.0.0.1;dbname=nba', 'root', '');

	// .... CRUD

} catch (PDOException $e) {
  print "Error!: ".$e->getMessage()."</br>";
} finally {
  $stmt = null; // 释放查询结果
  $dbh = null;  // 关闭连接
}
```

### MySQLi

```php
class foo_mysqli extends mysqli
{
  public function __construct($host, $user, $pass, $db)
  {
    parent::__construct($host, $user, $pass, $db);

    if (mysqli_connect_error()) {
      die('Connect Error ('.mysqli_connect_errno().') '
          .mysqli_connect_error());
    }
  }
}

$db = new foo_mysqli('127.0.0.1', 'root', '', 'nba');
echo 'Success... '.$db->host_info."\n";

$db->autocommit(true); // 开启自动提交

//    $db->begin_transaction(MYSQLI_TRANS_START_READ_ONLY); 开启只读事务
$db->begin_transaction(MYSQLI_TRANS_START_READ_WRITE);   // 开启读写事

// .... CRUD

$db->commit();            // 事务提交

$db->close();             // 关闭连接
```
