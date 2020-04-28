## 使用SQL需要考虑的因素

- 获得结果集所需访问的数据量
- 定义结果集所需的查询条件
- 结果集的大小
- 获得结果集所涉及的表的数量
- 同时修改这些数据用户的多少

## 优化的具体操作

1、在表中建立索引，优先考虑where、group by使用到的字段。

2、尽量避免使用select *，返回无用的字段会降低查询效率。如下：SELECT * FROM t 优化方式：使用具体的字段代替*，只返回使用到的字段。

3、尽量避免使用in 和not in，会导致数据库引擎放弃索引进行全表扫描。如下：

SELECT * FROM t WHERE id IN (2,3)

SELECT * FROM t1 WHERE username IN (SELECT username FROM t2)

优化方式：如果是连续数值，可以用between代替。如下：

SELECT * FROM t WHERE id BETWEEN 2 AND 3

如果是子查询，可以用exists代替。如下：

SELECT * FROM t1 WHERE EXISTS (SELECT * FROM t2 WHERE t1.username = t2.username)

4、尽量避免使用or，会导致数据库引擎放弃索引进行全表扫描。如下：

SELECT * FROM t WHERE id = 1 OR id = 3

优化方式：可以用union代替or。如下：

SELECT * FROM t WHERE id = 1
UNION
SELECT * FROM t WHERE id = 3

（PS：如果or两边的字段是同一个，如例子中这样。貌似两种方式效率差不多，即使union扫描的是索引，or扫描的是全表）

5、尽量避免在字段开头模糊查询，会导致数据库引擎放弃索引进行全表扫描。如下：

SELECT * FROM t WHERE username LIKE '%li%'

优化方式：尽量在字段后面使用模糊查询。如下：

SELECT * FROM t WHERE username LIKE 'li%'

6、尽量避免进行null值的判断，会导致数据库引擎放弃索引进行全表扫描。如下：

SELECT * FROM t WHERE score IS NULL

优化方式：可以给字段添加默认值0，对0值进行判断。如下：

SELECT * FROM t WHERE score = 0

7、尽量避免在where条件中等号的左侧进行表达式、函数操作，会导致数据库引擎放弃索引进行全表扫描。如下：

SELECT * FROM t2 WHERE score/10 = 9

SELECT * FROM t2 WHERE SUBSTR(username,1,2) = 'li'

优化方式：可以将表达式、函数操作移动到等号右侧。如下：

SELECT * FROM t2 WHERE score = 10*9

SELECT * FROM t2 WHERE username LIKE 'li%'