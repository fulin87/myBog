---
layout: post
title:  "mysql中使用explain来分析Sql语句的性能"
date:   2016-09-23 13:07:11 +0800
categories: 存储
---


> SQL优化的基本思路就是，先找到性能差的语句，然后找到性能差的原因，然后优化。当我们找到目标SQL之后，可以使用explain来分析SQL语句的执行过程，说白了就是看看这个语句的执行计划，找到性能瓶颈。

### 语法

	explain <语句>  //explain是专门分析select等查询语句的执行计划的

### 例子

 * 执行以下sql语句，看一下它的执行计划(*这是一个比较复杂的查询了，比较典型*)

		EXPLAIN 
			SELECT
				o.order_id,
				o.order_sn,
				o.parent_order_sn,
				o.outer_id,
				e.pay_work_code,
				o.add_time,
				o.order_source,
				p.product_id,
				p.product_num,
				p.product_name,
				p.sell_num,
				p.sell_price,
				t.cacle_time,
				t.order_id
			FROM
				gshop_order o
			LEFT JOIN gshop_order_product p ON o.order_id = p.order_id
			LEFT JOIN gshop_order_ext e ON o.order_id = e.order_id
			LEFT JOIN (
				SELECT
					g1.order_id,
					g1.order_sn,
					g1.parent_order_sn,
					g1.outer_id,
					g2.from_status,
					g2.add_time AS cacle_time,
					g2.to_status,
					count(*) AS nums
				FROM
					gshop_order g1
				LEFT JOIN gshop_order_log g2 ON g1.order_id = g2.order_id
				WHERE
					g1.order_belong = 3
				AND g1.order_status = 13
				AND g1.sfv_download = 0
				AND (
					g1.order_source = 22
					OR g1.order_source = 26
				)
				AND g2.from_status = 5
				AND g2.to_status = 13
				AND g2.add_time > 1472610864
				GROUP BY
					g1.order_id
			) t ON o.order_id = t.order_id
			WHERE
				t.order_id IS NOT NULL
			ORDER BY
				t.cacle_time DESC

 * 执行结果如下：

		+----+-------------+------------+--------+---------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------+---------+----------------------+------+-----------------------------+
		| id | select_type | table      | type   | possible_keys                                                                                                                                                 | key         | key_len | ref                  | rows | Extra                       |
		+----+-------------+------------+--------+---------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------+---------+----------------------+------+-----------------------------+
		|  1 | PRIMARY     | <derived2> | ALL    | NULL                                                                                                                                                          | NULL        | NULL    | NULL                 |    2 | Using where; Using filesort |
		|  1 | PRIMARY     | o          | eq_ref | PRIMARY                                                                                                                                                       | PRIMARY     | 4       | t.order_id           |    1 | NULL                        |
		|  1 | PRIMARY     | p          | ref    | idx_orderid                                                                                                                                                   | idx_orderid | 4       | t.order_id           |    1 | NULL                        |
		|  1 | PRIMARY     | e          | eq_ref | order_id                                                                                                                                                      | order_id    | 4       | t.order_id           |    1 | NULL                        |
		|  2 | DERIVED     | g1         | ref    | PRIMARY,idx_userid,idx_ordersn,idx_addtime_status,idx_areanumber,idx_couponid,idx_parentorderid,idx_parentordersn,idx_paytime_payid,idx_shippingsn,idx_status | idx_status  | 1       | const                |    2 | Using where                 |
		|  2 | DERIVED     | g2         | ref    | idx_orderid                                                                                                                                                   | idx_orderid | 4       | gshop_db.g1.order_id |    1 | Using where                 |
		+----+-------------+------------+--------+---------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------+---------+----------------------+------+-----------------------------+
		6 rows in set

 * explain 结果详解
	
	看到	explain 出来的结果，相信初学者应该已经懵逼了，面对这种情况，千万不能慌，要沉住气，要相信 *"我们的宇宙是存在规律的，并且规律是可以被认识的"*，其实就是说，所有的技术，只要认真学，都是可以学会的，下面对explain结果的每一个字段的含义进行解释

		id
			SQL执行顺序的标识，执行顺序是从大到小，例子中最先执行的是id为2的，也就是说是从下往上执行的。
	
		select_type
			就是select的类型,有这么几种：
			SIMPLE ： 简单SELECT(不使用UNION或子查询等）
			primary ： 最外层的select，在有子查询的语句中，最外层的select就是primary
			union : union语句的第二个或者说是后面那一个
			DEPENDENT UNION : UNION中的第二个或后面的SELECT语句，取决于外面的查询
			UNION RESULT : UNION的结果
			SUBQUERY : 子查询中的第一个SELECT
			DEPENDENT SUBQUERY : 子查询中的第一个SELECT，取决于外面的查询
			DERIVED : 派生表的SELECT(FROM子句的子查询)
	
		table
			显示这一行的数据是关于哪张表的.有时不是真实的表名字,看到的derivedx(x是个数字,我的理解是第几步执行的结果)
		
		type
			这列很重要,显示了连接使用了哪种类别,有无使用索引.从最好到最差的连接类型为const、eq_reg、ref、range、indexhe和ALL
			system : 这是const联接类型的一个特例。表仅有一行满足条件.
			const : 表最多有一个匹配行，它将在查询开始时被读取。因为仅有一行，在这行的列值可被优化器剩余部分认为是常数。const表很快，因为它们只读取一次！const用于用常数值比较PRIMARY KEY或UNIQUE索引的所有部分时
			eq_ref : 对于每个来自于前面的表的行组合，从该表中读取一行。这可能是最好的联接类型，除了const类型。它用在一个索引的所有部分被联接使用并且索引是UNIQUE或PRIMARY KEY。eq_ref可以用于使用= 操作符比较的带索引的列。比较值可以为常量或一个使用在该表前面所读取的表的列的表达式。
			ref : 对于每个来自于前面的表的行组合，所有有匹配索引值的行将从这张表中读取。如果联接只使用键的最左边的前缀，或如果键不是UNIQUE或PRIMARY KEY（换句话说，如果联接不能基于关键字选择单个行的话），则使用ref。如果使用的键仅仅匹配少量行，该联接类型是不错的。ref可以用于使用=或<=>操作符的带索引的列。
			ref_or_null : 该联接类型如同ref，但是添加了MySQL可以专门搜索包含NULL值的行。在解决子查询中经常使用该联接类型的优化。
			index_merge : 该联接类型表示使用了索引合并优化方法。在这种情况下，key列包含了使用的索引的清单，key_len包含了使用的索引的最长的关键元素。
			unique_subquery : 该类型替换了下面形式的IN子查询的ref：
				value IN (SELECT primary_key FROM single_table WHERE some_expr)
				unique_subquery是一个索引查找函数，可以完全替换子查询，效率更高。
			index_subquery : 该联接类型类似于unique_subquery。可以替换IN子查询
			range : 只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引。key_len包含所使用索引的最长关键元素。在该类型中ref列为NULL。当使用=、<>、>、>=、<、<=、IS NULL、<=>、BETWEEN或者IN操作符，用常量比较关键字列时，可以使用range
			index : 该联接类型与ALL相同，除了只有索引树被扫描。这通常比ALL快，因为索引文件通常比数据文件小。当查询只使用作为单索引一部分的列时，MySQL可以使用该联接类型。
			ALL ： 对于每个来自于先前的表的行组合，进行完整的表扫描。如果表是第一个没标记const的表，这通常不好，并且通常在它情况下很差。通常可以增加更多的索引而不要使用ALL，使得行能基于前面的表中的常数值或列值被检索出。
	
		possible_keys
			possible_keys列指出MySQL能使用哪个索引在该表中找到行。注意，该列完全独立于EXPLAIN输出所示的表的次序。这意味着在possible_keys中的某些键实际上不能按生成的表次序使用。如果该列是NULL，则没有相关的索引。在这种情况下，可以通过检查WHERE子句看是否它引用某些列或适合索引的列来提高你的查询性能。如果是这样，创造一个适当的索引并且再次用EXPLAIN检查查询
	
		key
			key列显示MySQL实际决定使用的键（索引）。如果没有选择索引，键是NULL。要想强制MySQL使用或忽视possible_keys列中的索引，在查询中使用FORCE INDEX、USE INDEX或者IGNORE INDEX。
	
		key_len
			key_len列显示MySQL决定使用的键长度。如果键是NULL，则长度为NULL。使用的索引的长度。在不损失精确性的情况下，长度越短越好
	
		ref
			ref列显示使用哪个列或常数与key一起从表中选择行。
	
		rows
			rows列显示MySQL认为它执行查询时必须检查的行数。
	
		Extra
			该列包含MySQL解决查询的详细信息，下面是详细信息
			Distinct ： 一旦MYSQL找到了与行相联合匹配的行，就不再搜索了
			Not exists ： MYSQL优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行就不再搜索了
			Range checked for each Record（index map:#）： 没有找到理想的索引，因此对于从前面表中来的每一个行组合，MYSQL检查使用哪个索引，并用它来从表中返回行。这是使用索引的最慢的连接之一
			Using filesort www.2cto.com ： 看到这个的时候，查询就需要优化了。MYSQL需要进行额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来排序全部行
			Using index ： 列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候
			Using temporary ： 看到这个的时候，查询需要优化了。这里，MYSQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上
			Using where ： 使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index，这就会发生，或者是查询有问题
	
 * 本例的执行计划解读

  	sql语句的执行是按照执行计划从下往上执行的，从执行计划可以看出，最先执行的是表连接中的查询，生成临时表与其他表连接查询，其他表都走的是索引，唯独临时表使用的是全表扫描。