问题发现
我认为一条很简单的SQL然后跑了很久，明明我已经都建立相应的索引，逻辑也不需要优化。

复制代码
SELECT a.custid, b.score, b.xcreditscore, b.lrscore
FROM (
    SELECT DISTINCT custid
    FROM sync.`credit_apply`
    WHERE SUBSTR(createtime, 1, 10) >= '2019-12-15'
        AND rejectrule = 'xxxx'
) a
    LEFT JOIN (
        SELECT *
        FROM sync.`credit_creditchannel`
    ) b
    ON a.custid = b.custid;
复制代码
查看索引状态：

credit_apply表

复制代码
mysql> show index from sync.`credit_apply`;
+--------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table        | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+--------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| credit_apply |          0 | PRIMARY  |            1 | applyId     | A         |     1468496 | NULL     | NULL   |      | BTREE      |         |               |
| credit_apply |          1 | index2   |            1 | custId      | A         |      666338 | NULL     | NULL   |      | BTREE      |         |               |
| credit_apply |          1 | index2   |            2 | createTime  | A         |     1518231 | NULL     | NULL   |      | BTREE      |         |               |
+--------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
复制代码
或者

复制代码
CREATE TABLE `credit_apply` (
  `applyId` bigint(20) NOT NULL AUTO_INCREMENT,
  `custId` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL,
  `ruleVersion` int(11) NOT NULL DEFAULT '1',
  `rejectRule` varchar(128) COLLATE utf8mb4_unicode_ci DEFAULT 'DP0000',
  `status` tinyint(4) NOT NULL DEFAULT '0',
  `extra` text COLLATE utf8mb4_unicode_ci,
  `createTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updateTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `mobile` varchar(128) COLLATE utf8mb4_unicode_ci DEFAULT '',
  PRIMARY KEY (`applyId`) USING BTREE,
  KEY `index2` (`custId`,`createTime`)
) ENGINE=InnoDB AUTO_INCREMENT=1567035 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
复制代码


sync.`credit_creditchannel`表

复制代码
mysql> show index from sync.`credit_creditchannel` ;
+----------------------+------------+-----------------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table                | Non_unique | Key_name                    | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+----------------------+------------+-----------------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| credit_creditchannel |          0 | PRIMARY                     |            1 | recId       | A         |      450671 | NULL     | NULL   |      | BTREE      |         |               |
| credit_creditchannel |          1 | nationalId_custid           |            1 | nationalId  | A         |      450770 | NULL     | NULL   |      | BTREE      |         |               |
| credit_creditchannel |          1 | nationalId_custid           |            2 | custId      | A         |      450770 | NULL     | NULL   | YES  | BTREE      |         |               |
| credit_creditchannel |          1 | credit_creditchannel_custId |            1 | custId      | A         |      450770 |       10 | NULL   | YES  | BTREE      |         |               |
+----------------------+------------+-----------------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
复制代码
 或者

复制代码
CREATE TABLE `credit_creditchannel` (
  `recId` bigint(20) NOT NULL AUTO_INCREMENT,
  `nationalId` varchar(128) NOT NULL DEFAULT '',
  `identityType` varchar(3) NOT NULL DEFAULT '',
  `brief` mediumtext,
  `score` decimal(10,4) NOT NULL DEFAULT '0.0000',
  `npaCode` varchar(128) NOT NULL DEFAULT '',
  `basic` mediumtext,
  `createTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updateTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `request` mediumtext,
  `custId` varchar(128) DEFAULT '',
  `xcreditScore` decimal(10,4) DEFAULT '0.0000',
  `queryTime` varchar(24) DEFAULT '',
  `lrScore` decimal(10,4) DEFAULT '0.0000',
  PRIMARY KEY (`recId`) USING BTREE,
  KEY `nationalId_custid` (`nationalId`,`custId`),
  KEY `credit_creditchannel_custId` (`custId`(10))
) ENGINE=InnoDB AUTO_INCREMENT=586557 DEFAULT CHARSET=utf8 
复制代码
我们都可以看到相应的索引。以现在简单的sql逻辑理论上走custid这个索引就好了

 

解释函数explain
复制代码
mysql> explain SELECT a.custid, b.score, b.xcreditscore, b.lrscore FROM(
SELECT DISTINCT custid FROM sync.`credit_apply` WHERE SUBSTR(createtime, 1, 10) >= '2019-12-15' AND rejectrule = 'xxx') a
LEFT JOIN 
(select * from sync.`credit_creditchannel`) b
ON a.custid = b.custid;
+----+-------------+----------------------+------------+-------+---------------+--------+---------+------+---------+----------+----------------------------------------------------+
| id | select_type | table                | partitions | type  | possible_keys | key    | key_len | ref  | rows    | filtered | Extra                                              |
+----+-------------+----------------------+------------+-------+---------------+--------+---------+------+---------+----------+----------------------------------------------------+
|  1 | PRIMARY     | <derived2>           | NULL       | ALL   | NULL          | NULL   | NULL    | NULL |  158107 |   100.00 | NULL                                               |
|  1 | PRIMARY     | credit_creditchannel | NULL       | ALL   | NULL          | NULL   | NULL    | NULL |  450770 |   100.00 | Using where; Using join buffer (Block Nested Loop) |
|  2 | DERIVED     | credit_apply         | NULL       | index | index2        | index2 | 518     | NULL | 1581075 |    10.00 | Using where                                        |
+----+-------------+----------------------+------------+-------+---------------+--------+---------+------+---------+----------+----------------------------------------------------+
3 rows in set (0.06 sec)
复制代码
如何去看我们的SQL是否走索引？

我们只需要注意一个最重要的type 的信息很明显的提现是否用到索引：

type结果

type结果值从好到坏依次是：

system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

一般来说，得保证查询至少达到range级别，最好能达到ref，否则就可能会出现性能问题。

possible_keys：sql所用到的索引

key：显示MySQL实际决定使用的键（索引）。如果没有选择索引，键是NULL

rows: 显示MySQL认为它执行查询时必须检查的行数。


分析：

我们的credit_creditchannel是ALL，而possible_keys是NULL索引在查询该表的时候并没有用到索引怪不得这么慢！！！！！！！！！

 

分析和搜索解决办法
换着法的改sql也没用；换着群问大神也没用；各种搜索引擎搜才总算有点思路。 

索引用不上的原因可能是字符集和排序规则不相同。

于是看了了两张表的字符集和两张表这个字段的字符集以及排序规则:



修改数据库和表的字符集
alter database sync default character set utf8mb4;//修改数据库的字符集
alter table sync.credit_creditchannel default character set utf8mb4;//修改表的字符集
修改表排序规则

alter table sync.`credit_creditchannel` convert to character set utf8mb4 COLLATE utf8mb4_unicode_ci;
 

由于数据库中的数据表和表字段的字符集和排序规则不统一，批量修改脚本如下：

1. 修改指定数据库中所有varchar类型的表字段的字符集为ut8mb4，并将排序规则修改为utf8_unicode_ci

复制代码
SELECT CONCAT('ALTER TABLE `', table_name, '` MODIFY `', column_name, '` ', DATA_TYPE, '(', CHARACTER_MAXIMUM_LENGTH, ') CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci', CASE 
        WHEN IS_NULLABLE = 'NO' THEN ' NOT NULL'
        ELSE ''
    END, ';')
FROM information_schema.COLUMNS
WHERE (TABLE_SCHEMA = 'databaseName'
    AND DATA_TYPE = 'varchar'
    AND (CHARACTER_SET_NAME != 'utf8mb4'
        OR COLLATION_NAME != 'utf8mb4_unicode_ci'));
复制代码
2. 修改指定数据库中所有数据表的字符集为UTF8，并将排序规则修改为utf8_general_ci　

SELECT CONCAT('ALTER TABLE ', table_name, ' CONVERT TO CHARACTER SET  utf8mb4 COLLATE utf8mb4_unicode_ci;')
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'sync_rs'
 

 

explain 查看是否用到了索引

复制代码
mysql> explain SELECT a.custid, b.score, b.xcreditscore, b.lrscore FROM(
SELECT DISTINCT custid FROM sync.`credit_apply` WHERE SUBSTR(createtime, 1, 10) >= '2019-12-15' AND rejectrule = 'xxx') a
LEFT JOIN 
(select * from sync.`credit_creditchannel`) b
ON a.custid = b.custid;
+----+-------------+----------------------+------------+-------+-----------------------------+-----------------------------+---------+----------+---------+----------+-------------+
| id | select_type | table                | partitions | type  | possible_keys               | key                         | key_len | ref      | rows    | filtered | Extra       |
+----+-------------+----------------------+------------+-------+-----------------------------+-----------------------------+---------+----------+---------+----------+-------------+
|  1 | PRIMARY     | <derived2>           | NULL       | ALL   | NULL                        | NULL                        | NULL    | NULL     |  146864 |   100.00 | NULL        |
|  1 | PRIMARY     | credit_creditchannel | NULL       | ref   | credit_creditchannel_custId | credit_creditchannel_custId | 43      | a.custid |       1 |   100.00 | Using where |
|  2 | DERIVED     | credit_apply         | NULL       | index | index2                      | index2                      | 518     | NULL     | 1468644 |    10.00 | Using where |
+----+-------------+----------------------+------------+-------+-----------------------------+-----------------------------+---------+----------+---------+----------+-------------+
复制代码
 

就是这样！！！！

补充大全：
可以看到结果中包含10列信息，分别为

id、select_type、table、type、possible_keys、key、key_len、ref、rows、Extra
对应的简单描述如下：

id: select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序===id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行
select_type: 表示查询的类型。用于区别普通查询、联合查询、子查询等的复杂查询。
table: 输出结果集的表 显示这一步所访问数据库中表名称（显示这一行的数据是关于哪张表的），有时不是真实的表名字，可能是简称，例如上面的e，d，也可能是第几步执行的结果的简称
partitions:匹配的分区
type:对表访问方式，表示MySQL在表中找到所需行的方式，又称“访问类型”。
possible_keys:表示查询时，可能使用的索引
key:表示实际使用的索引
key_len:索引字段的长度
ref:列与索引的比较
rows:扫描出的行数(估算的行数)
filtered:按表条件过滤的行百分比
Extra:执行情况的描述和说明
挑选一些重要信息详细说明：

select_type
SIMPLE 简单的select查询，查询中不包含子查询或者UNION
PRIMARY 查询中若包含任何复杂的子部分，最外层查询则被标记为PRIMARY
SUBQUERY 在SELECT或WHERE列表中包含了子查询
DERIVED 在FROM列表中包含的子查询被标记为DERIVED（衍生），MySQL会递归执行这些子查询，把结果放在临时表中
UNION 若第二个SELECT出现在UNION之后，则被标记为UNION：若UNION包含在FROM子句的子查询中，外层SELECT将被标记为：DERIVED
UNION RESULT 从UNION表获取结果的SELECT
 

type
mysql找到数据行的方式，效率排名
NULL > system > const > eq_ref > ref > range > index > all
***一般来说，得保证查询至少达到range级别，最好能达到ref。

system 表只有一行记录（等于系统表），这是const类型的特列，平时不会出现，这个也可以忽略不计
const 通过索引一次就找到了，const用于比较primary key 和 unique key，因为只匹配一行数据，所以很快。如果将主键置于where列表中，mysql就能将该查询转换为一个常量
eq_ref 唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键索引和唯一索引 区别于const eq_ref用于联表查询的情况
ref 非唯一索引扫描，返回匹配某个单独值的所有行，本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，他可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体
range 只检索给定范围的行，使用一个索引来选择行，一般是在where中出现between、<、>、in等查询，范围扫描好于全表扫描，因为他只需要开始于索引的某一点，而结束于另一点，不用扫描全部索引
index Full Index Scan，Index与All区别为index类型只遍历索引树。通常比All快，因为索引文件通常比数据文件小。也就是说，虽然all和index都是读全表，但是index是从索引中读取的，而all是从硬盘读取的
ALL Full Table Scan,将遍历全表以找到匹配的行
 

possible_keys
指出mysql能使用哪个索引在表中找到记录，查询涉及到的字段若存在索引，则该索引被列出，但不一定被查询使用（该查询可以利用的索引，如果没有任何索引显示null）
实际使用的索引，如果为NULL，则没有使用索引。（可能原因包括没有建立索引或索引失效）
查询中若使用了覆盖索引（select 后要查询的字段刚好和创建的索引字段完全相同），则该索引仅出现在key列表中 possible_keys为null

 

key
key列显示mysql实际决定使用的索引，必然包含在possible_keys中。如果没有选择索引，键是NULL。想要强制使用或者忽视possible_keys列中的索引，在查询时指定FORCE INDEX、USE INDEX或者IGNORE index

key_len
表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度，在不损失精确性的情况下，长度越短越好。key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的。

ref
显示索引的那一列被使用了，如果可能的话，最好是一个常数。哪些列或常量被用于查找索引列上的值。

rows
根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数，也就是说，用的越少越好

 

extra
包含不适合在其他列中显式但十分重要的额外信息

Using Index:表示相应的select操作中使用了覆盖索引（Covering Index），避免访问了表的数据行，效率不错。如果同时出现using where，表明索引被用来执行索引键值的查找；如果没有同时出现using where，表明索引用来读取数据而非执行查找动作。
Using where:不用读取表中所有信息，仅通过索引就可以获取所需数据，这发生在对表的全部的请求列都是同一个索引的部分的时候，表示mysql服务器将在存储引擎检索行后再进行过滤
Using temporary：表示MySQL需要使用临时表来存储结果集，常见于排序和分组查询，常见 group by ; order by
Using filesort：当Query中包含 order by 操作，而且无法利用索引完成的排序操作称为“文件排序”
Using join buffer：表明使用了连接缓存,比如说在查询的时候，多表join的次数非常多，那么将配置文件中的缓冲区的join buffer调大一些。
Impossible where：where子句的值总是false，不能用来获取任何元组
Select tables optimized away：这个值意味着仅通过使用索引，优化器可能仅从聚合函数结果中返回一行
No tables used：Query语句中使用from dual 或不含任何from子句
以上两种信息表示mysql无法使用索引

using filesort ：表示mysql会对结果使用一个外部索引排序，而不是从表里按索引次序读到相关内容，可能在内存或磁盘上排序。mysql中无法利用索引完成的操作称为文件排序
using temporary: 使用了用临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于排序order by和分组查询group by。
 

大多数人都以为是才智成就了科学家，他们错了，是品格。---爱因斯坦
