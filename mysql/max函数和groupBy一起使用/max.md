# [MAX函数和GROUP BY 语句一起使用的一个误区](https://www.cnblogs.com/exmyth/p/3986680.html)

https://www.cnblogs.com/exmyth/p/3986680.html

使用MAX 函数和 GROUP 的时候会有不可预料的数据被SELECT 出来。
下面举个简单的例子：
想知道每个SCOREID 的 数学成绩最高的分数。

表信息：
/*DDL Information For - test.lkscore*/
\--------------------------------------

Table  Create Table                                 
------- -----------------------------------------------------------------------------
lkscore CREATE TABLE `lkscore` (                           
      `scoreid` int(11) DEFAULT NULL,                      
      `chinese` int(11) DEFAULT '0',                       
      `math` int(11) DEFAULT '0',                        
      KEY `fk_class` (`scoreid`),                        
      CONSTRAINT `fk_class` FOREIGN KEY (`scoreid`) REFERENCES `lkclass` (`id`) 
     ) ENGINE=InnoDB DEFAU

```mysql
select * from lkscore;
```

### query result(12 records)

| scoreid | chinese | math |
| ------- | ------- | ---- |
| 1       | 90      | 80   |
| 2       | 100     | 99   |
| 3       | 29      | 98   |
| 4       | 87      | 79   |
| 5       | 89      | 99   |
| 1       | 49      | 98   |
| 3       | 98      | 56   |
| 2       | 76      | 88   |
| 2       | 80      | 90   |
| 3       | 90      | 70   |
| 1       | 90      | 90   |
| 1       | 67      | 90   |

错误的SELECT

```mysql
select scoreid,chinese,max(math) max_math from lkscore group by scoreid;
```

### query result(5 records)

| scoreid | chinese | max_math |
| ------- | ------- | -------- |
| 1       | 90      | 98       |
| 2       | 100     | 99       |
| 3       | 29      | 98       |
| 4       | 87      | 79       |
| 5       | 89      | 99       |

上面的90明显不对。

方法一：

```mysql
select scoreid,chinese,math max_math from 
(
select * from lkscore order by math desc
) T 
group by scoreid;
```

### query result(5 records)

| scoreid | chinese | max_math |
| ------- | ------- | -------- |
| 1       | 49      | 98       |
| 2       | 100     | 99       |
| 3       | 29      | 98       |
| 4       | 87      | 79       |
| 5       | 89      | 99       |


方法二：

```mysql
select * from lkscore a where a.math = (select max(math) from lkscore where scoreid = a.scoreid) order by scoreid asc;
```

### query result(5 records)

| scoreid | chinese | max_math |
| ------- | ------- | -------- |
| 1       | 49      | 98       |
| 2       | 100     | 99       |
| 3       | 29      | 98       |
| 4       | 87      | 79       |
| 5       | 89      | 99       |


这个也是用MAX函数，而且还用到了相关子查询。
我们来看一下这两个的效率如何：

```mysql
explain 
select scoreid,chinese,math max_math from (select * from lkscore order by math desc) T group by scoreid;
```

### query result(2 records)

| id   | select_type | table      | type | possible_keys | key    | key_len | ref    | rows | Extra                           |
| ---- | ----------- | ---------- | ---- | ------------- | ------ | ------- | ------ | ---- | ------------------------------- |
| 1    | PRIMARY     | <derived2> | ALL  | (NULL)        | (NULL) | (NULL)  | (NULL) | 12   | Using temporary; Using filesort |
| 2    | DERIVED     | lkscore    | ALL  | (NULL)        | (NULL) | (NULL)  | (NULL) | 12   | Using filesort                  |

很明显，有两个FULL TABLE SCAN。

```

explain 
select scoreid,chinese,math max_math from lkscore a where a.math = 
(select max(math) from lkscore where scoreid = a.scoreid) order by scoreid asc;
```

### query result(2 records)

| id   | select_type        | table   | type  | possible_keys | key      | key_len | ref       | rows | Extra       |
| ---- | ------------------ | ------- | ----- | ------------- | -------- | ------- | --------- | ---- | ----------- |
| 1    | PRIMARY            | a       | index | (NULL)        | fk_class | 5       | (NULL)    | 12   | Using where |
| 2    | DEPENDENT SUBQUERY | lkscore | ref   | fk_class      | fk_class | 5       | a.scoreid | 1    | Using where |



第二个就用了KEY,子查询里只扫描了一跳记录。

很明显。在这种情况下第二个比第一个效率高点。