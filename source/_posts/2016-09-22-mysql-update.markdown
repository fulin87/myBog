---
layout: post
title:  "mysql中sql优化的思路和原则"
date:   2016-09-22 13:07:11 +0800
categories: 存储
---


> 大部分系统的性能瓶颈都是IO操作，数据库的IO消耗是其中的重要组成部分，SQL语句性能的优化是很有必要的，如果一开始不注意，后期的改动成本是比较大的,下面是根据我自己的经验总结的一套sql优化的思路和方法，不一定是最好的，但是却真实的产生过效果。

### 入乎其内；出乎其外

 *为什么人类到今天为止对宇宙的认识都很有限？因为我们自己就在宇宙之中；为什么人类到今天为止对海洋的认识都很少？因为人类从来没有正真的进入过海洋！*

 一样的道理，认识一个事物，最有效的方式就是：先站在事物的外面观察观察它，然后进入事物的内部再研究它，这样才是最全面的。

 对于SQL来说，平时使用sql，编写sql语句就相当于站在sql的外面观察它，但是sql的内部机制和执行过程是怎样的呢？一个sql语句的执行过程，大致是这样的：

	8	SELECT  DISTINCT <select_list>
	1	FROM <left_table>
	3	<join_type>JOIN<right_table>
	2		ON<join_condition>
	4	WHERE <where_condition>
	5	GROUP BY<group_by_list>
	6	WITH{CUBE|ROLLUP}
	7	HAVING<having_condition>
	10	ORDER BY<order_by_list>
	11	LIMIT<limit_number>

下面是详细的执行过程

	1,FROM : 	对FROM的左边表和右边的表计算笛卡尔积，产生虚拟表VT1。
	2,ON : 		对虚拟表VT1进行ON筛选，只有那些符合<join_condition>的行才会被记录在虚拟表VT2中。
	3,JOIN : 	如果指定了OUTER JOIN (比如left join,right join),那么保留表中未匹配的行就会作为外部行添加到虚拟表VT2中，产生虚拟表VT3,from 字句中包含两个以上的表的话，就会对上一个join连接产生的结果VT3和下一个表重复执行步骤1-3这三个步骤，一直到处理完所有的表为止。
	4,WHERE : 	对虚拟表VT3进行WHERE条件过滤。只要符合<where-condition>的记录才会被插入到虚拟表VT4中。
	5,GROUP BY : 	根据group by字句中的列，对VT4中的记录进行分组操作，产生VT5。
	6,CUBE | ROLLUP: 对表VT5进行cube或者rollup操作，产生表VT6。
	7,HAVING：	对虚拟表VT6应用having过滤，只有符合<having-condition>的记录才会被 插入到虚拟表VT7中。
	8,SELECT： 	执行select操作，选择指定的列，插入到虚拟表VT8中。
	9,DISTINCT： 	对VT8中的记录进行去重。产生虚拟表VT9。
	10,ORDER BY: 	将虚拟表VT9中的记录按照<order_by_list>进行排序操作，产生虚拟表VT10。
	11,LIMIT：	取出指定行的记录，产生虚拟表VT11, 并将结果返回。


### SQL优化的思路

- 1.优化更需要优化的sql；
- 2.定位优化对象的性能瓶颈：优化前需了解查询的瓶颈是IO还是CPU可通过PROFILING很容易定位查询的瓶颈。
- 3.明确优化目标；
- 4.从Explain入手；
- 5.多使用profile；

### SQL优化的基本原则

 * 永远用小结果集驱动大结果集

	```From子句中sql解析顺序为从右向左，执行时会以最左边的表为基础表循环与右边表数据做笛卡尔积，所以以小结果集驱动能减少循环次数，从而减少对被驱动结果集的访问，从而减少被驱动表的锁定。```	
 * 尽可能在索引中完成排序

	```排序算法有两种：a.查出排序字段和行指针，排序，再通过行指针获得行数据所需列，返回结果集；b.取出所有排序列数据，在排序缓冲区中排完序直接返回结果集。索引排序是利用索引的有序性对数据排序的。```
 * 只取出子集需要的colums
 * 仅仅使用最有效的过滤条件
 * 尽可能避免复杂的Join和子查询

### 索引相关的总结 

> 什么是索引，可以把索引想象成数据库的目录，查找某一样东西之前，先在目录中确定好位置，直奔位置而去，这就是提高查询效率的原因。

 * 索引的好处

		提高数据检索效率，降低数据库的IO成本
		降低数据排序成本：要求排序字段和索引键字段一致
		降低数据分组成本：因为分组之前会先排序，同意如果分组字段与索引字段一致，会降低分组消耗的成本。

 * 索引的弊端

		索引是独立于基础数据的数据库对象，因此它会占用存储空间
		数据新增、更新会导致索引的同步更新，所以会增加数据新增、更新所消耗的成本

 * 否需要创建索引
	
		较为频繁的作为查询条件的字段需要创建索引
		唯一性太差的字段不适合单独创建索引，即使频繁作为查询条件
		更新非常频繁的字段不适合创建索引
		不会出现在where子句中的字段不要创建索引

 * 索引语法

	* 唯一索引
	
			ALTER TABLE tableName ADD UNIQUE indexName (column);
			CREATE UNIQUE INDEX indexName ON tableName (column);

	* 普通索引

			ALTER TABLE tableName ADD INDEX indexName(column);
			CREATE INDEX indexName ON tableName(column);

	* 主键索引
	
			ALTER TABLE tableName ADD PRIMARY KEY (column);

	* 全文索引

			ALTER TABLE tableName ADD FULLTEXT (column);

	* 组合索引

			ALTER TABLE tableName ADD INDEX indexName(col1,col2,...);