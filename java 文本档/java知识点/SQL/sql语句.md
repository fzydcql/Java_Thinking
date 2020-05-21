## 八种很坑的SQL错误用法

### 1.LIMIT语句

分页查询是最常用的场景之一，单也通常也是最容易出问题的地方。比如对于下面简单的语句，一般DBA想到的办法是在type，name，create_time字段上加组合索引。这样条件排序都能有效的利用到索引，性能迅速提升。

```
SELECT *
FROM operation
WHERE type = 'SQLStats'
      AND name = 'SlowLog'
ORDER BY create_time
LIMIT 1000,10;
```

但是当LIMIT子句变成“LIMIT 1000000,10”时，数据查询就会变得很慢，因为数据库并不知道第100000条记录从什么地方开始，即使有索引也需要从头计算一次。

所以在前端浏览翻页，或者大数据分批导出等场景，是可以将上一页的最大值当成参数作为查询条件的。SQL优化如下：

```
SELECT *
FROM operation
WHERE type = 'SQLStats'
AND name = 'SlowLog'
AND create_time > '2017-03-16 14:00:00'
ORDER BY create_time limit 10;
在新设计下查询基本固定，不会随着数据量的增长而发生变化。
```

### 2.隐式转换

SQL语句中查询变量和字段定义类型不匹配是一个常见的错误。比如：

```
SELECT *
FROM my_balance b
WHERE b.bpn = 14000000123
AND b.isverified IS NULL;
其中字段bpn的定义为varchar(20)，MYSQL的策略是将字符串转换成数字之后再比较。函数作用于表字段，索引失效。
上述错误可能会是自己集成的框架自动填入的参数。
```

### 3.关联更新，删除

虽然MySQL5.6引入了物化特性，但需要特别注意它目前仅仅针对查询语句的优化。对于更新或删除需要手动重写成JOIN。比如：

```
UPDATE operation o
SET status = 'applying'
WHERE o.id IN (SELECT id
              FROM   (SELECT o.id,
                     o.status
              FROM   operation o
              WHERE  o.group = 123
                     AND o.status NOT in("done")
                     ORDER BY o.parent,
                              o.id
                     LIMIT 1) t);
```

执行计划：

| id              | select_type                                  | table          | type                 | possible_keys           | key                      | key_len        | ref               | rows            | Extra                                  |
| --------------- | -------------------------------------------- | -------------- | -------------------- | ----------------------- | ------------------------ | -------------- | ----------------- | --------------- | -------------------------------------- |
| 1<br />2<br />3 | PRIMARY<br />DEPENDENT SUBQUERY<br />DERIVED | o<br /><br />o | index<br /><br />ref | <br /><br />idx_2,idx_5 | PRIMARY<br /><br />idx_5 | 8<br /><br />8 | <br /><br />const | 24<br /><br />1 | Using where;Using temporary Impossible |

优化重写join

```
UPDATE operation o
        JOIN (SELECT o.id,
                          o.status
                     FROM operation o
                     WHERE o.group = 123
                           AND o.status NOT IN ("done")
                           ORDER BY o.parent，
                                    o.id
                           LIMIT 1) t
               ON o.id = t.id
SET status = 'applying'
```

### 4.混合排序

MySQL不能利用索引进行混合排序。但是在某些场景，还是有机会使用特殊办法提升性能的。

```
SELECT * 
FROM my_order o
     JOIN my_appraise a ON a.orderid = o.id
ORDER BY a.is_reply ASC,
         a.appraise_time DESC
LIMIT 0,20
```

根据字段is_reply只有0和1两种状态，我们按照下面的方法重写后，执行时间从1.58秒降低到2毫秒。

```
SELECT *
FROM ((SELECT *
       FROM my_order o
            JOIN my_appraise a
              ON a.orderid = o.id
                 AND is_reply = 0
        OREDR BY appraise_time DESC
        LIMIT 0,20)
        UNION ALL
        (SELECT *
         FROM  My_order o
          JOIN my_appraise a
              ON a.orderid = o.id
                 AND is_reply = 0
        OREDR BY appraise_time DESC
        LIMIT 0,20)) t
ORDER BY a.is_reply ASC,
       appraise_time DESC
LIMIT 20   
```

### 5.EXISTS语句

