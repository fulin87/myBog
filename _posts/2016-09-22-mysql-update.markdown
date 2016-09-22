---
layout: post
title:  "mysql中sql优化的思路和原则"
categories: mysql
---

> 大部分系统的性能瓶颈都是IO操作，数据库的IO消耗是其中的重要组成部分，SQL语句性能的优化是很有必要的，如果一开始不注意，后期的改动成本是比较大的

### SQL优化的思路

	1.优化更需要优化的sql；
	2.定位优化对象的性能瓶颈：优化前需了解查询的瓶颈是IO还是CPU可通过PROFILING很容易定位查询的瓶颈。
	3.明确优化目标；
	4.从Explain入手；
	5.多使用profile；

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