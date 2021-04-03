---
title: MySQL可以用到like索引吗
date: 2021-04-03 15:39:40
tags: MySQL
---

MySQL的like查询，可以用到索引吗？这是一个在面试中会被经常问到的问题。大部分情况下，大部分面试官希望听到的答案是，百分号在左边的情况下，是不会用到索引的，百分号在右边的情况，可以用到索引。那么，事实真的是这样吗？

我们先看两个SQL，已知member表有400万条数据，其中mobile是索引字段。

SQL1：

```sql
select * from member where mobile like '%1300%' limit 10000,10;
```

SQL2：

```sql
select mobile from member where mobile like '%1300%' limit 10000,10;
```

两条SQL会有运行速度上的差异吗？我们直接看结果

```sql
select * from member where mobile like '%1300%' limit 10000,10;
10 rows in set (5.97 sec)
```

```sql
select mobile from member where mobile like '%1300%' limit 10000,10;
10 rows in set (1.67 sec)
```

SQL1查询用时5.97秒 SQL2用时1.67秒，为什么看似相同的两个SQL，查询速度差别如此明显？

我们来看一下这两条SQL的查询

```sql
mysql> explain select * from member where mobile like '%1300%' limit 10000,10;
+----+-------------+--------+------+---------------+------+---------+------+---------+-------------+
| id | select_type | table  | type | possible_keys | key  | key_len | ref  | rows    | Extra       |
+----+-------------+--------+------+---------------+------+---------+------+---------+-------------+
|  1 | SIMPLE      | member | ALL  | NULL          | NULL | NULL    | NULL | 3947590 | Using where |
+----+-------------+--------+------+---------------+------+---------+------+---------+-------------+
1 row in set (0.00 sec)
```

```sql
mysql> explain select mobile from member where mobile like '%1300%' limit 10000,10;
+----+-------------+--------+-------+---------------+--------+---------+------+---------+--------------------------+
| id | select_type | table  | type  | possible_keys | key    | key_len | ref  | rows    | Extra                    |
+----+-------------+--------+-------+---------------+--------+---------+------+---------+--------------------------+
|  1 | SIMPLE      | member | index | NULL          | mobile | 62      | NULL | 3947590 | Using where; Using index |
+----+-------------+--------+-------+---------------+--------+---------+------+---------+--------------------------+
1 row in set (0.00 sec)
```

神奇的结果发生了，我们发现，SQL2使用到了索引查询。

这种情况，我们成为覆盖索引。当我们使用覆盖索引查询的时候，MySQL会直接在索引上进行查询的操作，就不会再进行回表操作。如此一来，速度大大提升。