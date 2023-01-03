# mysql 语句优化

## 总览

1. 优化一条慢查询sql最先需要做的是在表中添加索引。为`where`分句中使用到的字段添加索引能加速评估、过滤以及获得数据。为了避免浪费磁盘空间，项目中使用的索引应该小且能满足大多数查询。

   在使用外键和`join`的连表查询时，索引尤其重要。

2. 像那些使用了函数以及其他一下开销很大的语句需要对其调整、进行单独拆分。根据查询语句构成的不同，一个函数可能在结果集的每一行中进行调用，也有可能在整张表的每一行中进行调用，这会造成巨大的性能开销。

3. 对于全表扫描而言，尽量减小扫描的数量。数据量越大的表这点越重要

4. 周期性地使用`ANALYZE TABLE`语句来更新表的统计数据，这样的话优化器能获取需要的信息去构建高效的执行计划。

5. 学习调整技巧、索引技巧以及针对搜索引擎的配置参数

6. You can optimize single-query transactions for `InnoDB` tables, using the technique in [Section 8.5.3, “Optimizing InnoDB Read-Only Transactions”](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-ro-txn.html).

7. 避免将查询语句写得复杂难懂，尤其是优化器在某些情况下自动做了同样处理时。

8. 当一个语句不能轻松地由基础指导优化时，使用`explain` 观察其细节，并且调整索引、`where` 分句、`join`分句（当你掌握了一定的技能后，阅读`explain`可能是优化所有语句的第一步）

## where分句优化

你可能会重写查询语句，牺牲可读性来加快运算速度。但这是不必要的，因为mysql会做自动做类似的优化操作

* 去除不必要的括号

  ``` mysql	
  ((a AND b) AND c OR (((a AND b) AND (c AND d))))
  -> (a AND b AND c) OR (a AND b AND c AND d)
  ```

* 折叠常数

	``` mysql
  (b>=5 AND b=5) OR (b=6 AND 5=5) OR (b=7 AND 5=6)
	-> b=5 OR b=6
	```

* 使用了索引的常数表达式只被评估一次
* 
