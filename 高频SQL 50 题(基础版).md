# 高频 SQL 50 题（基础版）

## Day 1:

### [584. 寻找用户推荐人](https://leetcode.cn/problems/find-customer-referee/)

------

找出那些 **没有被** `id = 2` 的客户 **推荐** 的客户的姓名。

**示例 1：**

```sql
输入： 
Customer 表:
+----+------+------------+
| id | name | referee_id |
+----+------+------------+
| 1  | Will | null       |
| 2  | Jane | null       |
| 3  | Alex | 2          |
| 4  | Bill | null       |
| 5  | Zack | 1          |
| 6  | Mark | 2          |
+----+------+------------+
输出：
+------+
| name |
+------+
| Will |
| Jane |
| Bill |
| Zack |
+------+
```



**方法： 使用 <> (!=) 和 IS NULL **

有的人也许会非常直观地想到如下解法。

```sql
SELECT name FROM customer WHERE referee_Id <> 2;
```


然而，这个查询只会返回一个结果：Zach，尽管事实上有 4 个顾客都不是 Jane 推荐的（包括 Jane 她自己）。所有没有推荐人（referee_id 字段值为 NULL) 的全部都消失了。为什么?

MySQL 使用三值逻辑 —— TRUE, FALSE 和 UNKNOWN。任何与 NULL 值进行的比较都会与第三种值 UNKNOWN 做比较。这个“任何值”包括 NULL 本身！这就是为什么 MySQL 提供 IS NULL 和 IS NOT NULL 两种操作来对 NULL 特殊判断。

因此，在 WHERE 语句中我们需要做一个额外的条件判断 `referee_id IS NULL'。



1. **三值逻辑：** NULL在数据库中有两种情况，一是“未知”（unknown），二是“不适用”（inapplicable）。这种处理方式是为了处理某些属性不适用或未知的情况。
2. **为什么使用 `IS NULL`：** SQL采用三值逻辑，所以对NULL使用比较谓词（如 `= NULL`）的结果是unknown。NULL不是一个值，而是一个没有值的标记，因此不能使用比较谓词。SQL中的 `IS NULL` 应被理解为一个谓词，而不是两个独立的单词, 如果可以IS_NULL或许更合适。。重点理解： NULL不是值，所以不能对其使用谓词，如果使用谓词，结果是unknown。 可以认为它只是一个没有值的标记,而比较谓词只适用于比较值。因此对非值的NULL使用比较谓词是没有意义的
3. **误解和混淆：** 作者指出了一些可能导致误解的原因，比如将NULL误认为是一个值，以及由于 `IS NULL` 中有两个单词而产生的混淆。作者建议将 `IS NULL` 视为一个谓词。



解法 1：

```sql
SELECT name 
FROM Customer C 
WHERE C.referee_id <> 2 
OR C.referee_id 
IS NULL;

```

解法 2：

```sql
SELECT c.name 
FROM Customer c 
WHERE c.id 
NOT IN 
(SELECT c1.id 
 FROM Customer c1 
 WHERE c1.referee_id=2);

```

- `NOT IN` 是一个条件谓词，用于检查主查询的结果集中的值是否不在子查询的结果集中。
- 当使用 `NOT IN` 时，主查询的结果集中的值在子查询的结果集中找到匹配时，该行会被排除。
- `NOT IN` 的子查询通常返回一个包含单列值的结果集。

解法 3：

```sql
SELECT c.name 
FROM Customer c 
WHERE 
NOT EXISTS 
(SELECT c1.id 
 FROM Customer c1 
 WHERE c1.referee_id=2 
 and c.id=c1.id);

```

- `WHERE c1.referee_id = 2 AND c1.id = c.id`: 子查询的条件，它筛选出 `referee_id` 为 2 且 `id` 与主查询相匹配的行。
- 主查询中的 `WHERE NOT EXISTS (...)` 表示主查询的结果将包括那些子查询条件不成立的行。如果子查询返回了任何行（即，存在某个 `referee_id` 为 2 且 `id` 匹配的行），那么 `NOT EXISTS` 就为假，这样的行将不会被包括在结果中。





------

### 二. [595. 大的国家](https://leetcode.cn/problems/big-countries/)



如果一个国家满足下述两个条件之一，则认为该国是 **大国** ：

- 面积至少为 300 万平方公里（即，`3000000 km2`），或者
- 人口至少为 2500 万（即 `25000000`）

编写解决方案找出 **大国** 的国家名称、人口和面积。



**示例：**

```sql
输入：
World 表：
+-------------+-----------+---------+------------+--------------+
| name        | continent | area    | population | gdp          |
+-------------+-----------+---------+------------+--------------+
| Afghanistan | Asia      | 652230  | 25500100   | 20343000000  |
| Albania     | Europe    | 28748   | 2831741    | 12960000000  |
| Algeria     | Africa    | 2381741 | 37100000   | 188681000000 |
| Andorra     | Europe    | 468     | 78115      | 3712000000   |
| Angola      | Africa    | 1246700 | 20609294   | 100990000000 |
+-------------+-----------+---------+------------+--------------+
输出：
+-------------+------------+---------+
| name        | population | area    |
+-------------+------------+---------+
| Afghanistan | 25500100   | 652230  |
| Algeria     | 37100000   | 2381741 |
+-------------+------------+---------+
```



方法一：使用 WHERE 子句和 OR

```sql
SELECT name,population, area FROM World w WHERE w.area>=3000000 OR w.population>=25000000;
```

方法二：使用 WHERE 子句和 UNION
使用 or 会使索引会失效，在数据量较大的时候查找效率较低，通常建议使用 union 代替 or

```sql
SELECT w.name,w.population, w.area FROM World w 
        WHERE w.area>=3000000 
        UNION
SELECT w.name,w.population, w.area FROM World w 
        WHERE w.population>=25000000;
```

------



### 三. [1148. 文章浏览 I](https://leetcode.cn/problems/article-views-i/)



请查询出所有浏览过自己文章的作者

结果按照 `id` 升序排列。

**示例 1：**

```sql
输入：
Views 表：
+------------+-----------+-----------+------------+
| article_id | author_id | viewer_id | view_date  |
+------------+-----------+-----------+------------+
| 1          | 3         | 5         | 2019-08-01 |
| 1          | 3         | 6         | 2019-08-02 |
| 2          | 7         | 7         | 2019-08-01 |
| 2          | 7         | 6         | 2019-08-02 |
| 4          | 7         | 1         | 2019-07-22 |
| 3          | 4         | 4         | 2019-07-21 |
| 3          | 4         | 4         | 2019-07-21 |
+------------+-----------+-----------+------------+

输出：
+------+
| id   |
+------+
| 4    |
| 7    |
+------+
```

```python
import pandas as pd

def article_views(views: pd.DataFrame) -> pd.DataFrame:
    df=views[views['author_id']==views['viewer_id']]
    df.drop_duplicates(subset=['author_id'],inplace=True)
    df.sort_values(by=['author_id'],inplace=True)
    df.rename(columns={'author_id':'id'},inplace=True)
    df=df[['id']]
    return df
```



- 去重
  drop_duplicates() 函数
- 列重命名
  rename(columns={'author_id':'id'})
- 排序
  默认升序 sort_values(['id'])



```sql
SELECT DISTINCT v.author_id as id FROM Views v WHERE EXISTS 
      (SELECT v2.viewer_id FROM Views v2 
      WHERE v2.viewer_id=v.author_id) ORDER BY id ASC;
```





```sql
SELECT 
    DISTINCT author_id AS id 
FROM 
    Views 
WHERE 
    author_id = viewer_id 
ORDER BY 
    id 
```



-----------------------------------------------------



### 四. [1683. 无效的推文](https://leetcode.cn/problems/invalid-tweets/)



查询所有无效推文的编号（ID）。当推文内容中的字符数**严格大于** `15` 时，该推文是无效的。

以**任意顺序**返回结果表。

 

#### **1.示例：**

```sql
输入：
Tweets 表：
+----------+----------------------------------+
| tweet_id | content                          |
+----------+----------------------------------+
| 1        | Vote for Biden                   |
| 2        | Let us make America great again! |
+----------+----------------------------------+

输出：
+----------+
| tweet_id |
+----------+
| 2        |
+----------+
解释：
推文 1 的长度 length = 14。该推文是有效的。
推文 2 的长度 length = 32。该推文是无效的。
```



#### **2.SQL解答：**

```sql
SELECT 
  tweet_id 
FROM 
  tweets
WHERE 
  CHAR_LENGTH(content)>15;
```

1. **`char_length(str)`：**
   - **计算单位：** 字符
   - **字符定义：** 无论是汉字、数字还是字母，都被视为一个字符。
   - **例子：** 如果字符串中包含汉字，则每个汉字都被计为一个字符。比如，字符串 "你好123" 的 `char_length` 结果为 5。
2. **`length(str)`：**
   - **计算单位：** 字节
   - **编码方式：** 根据编码方式不同（例如，UTF-8 或 GBK），一个字符可能占用不同数量的字节。
   - **UTF-8 编码：** 一个汉字占用三个字节，一个数字或字母占用一个字节。
   - **GBK 编码：** 一个汉字占用两个字节，一个数字或字母占用一个字节。
   - **例子：** 如果字符串中包含汉字，则每个汉字可能占用三个字节（UTF-8）或两个字节（GBK）。比如，字符串 "你好123" 的 `length` 结果取决于编码方式。



#### **3.Pandas解答：**

```python
import pandas as pd

def invalid_tweets(tweets: pd.DataFrame) -> pd.DataFrame:
  is_valid=tweets['content'].str.len()>15
  df=tweets[is_valid]
  return df[['tweet_id']]
    
```



1. `str.len()` 方法: 计算字符串长度



## Day 2 

### [1378. 使用唯一标识码替换员工ID](https://leetcode.cn/problems/replace-employee-id-with-the-unique-identifier/)



展示每位用户的 **唯一标识码（unique ID ）**；如果某位员工没有唯一标识码，使用 null 填充即可。

你可以以 **任意** 顺序返回结果表。

返回结果的格式如下例所示。

 

#### **示例 1：**

```sql
输入：
Employees 表:
+----+----------+
| id | name     |
+----+----------+
| 1  | Alice    |
| 7  | Bob      |
| 11 | Meir     |
| 90 | Winston  |
| 3  | Jonathan |
+----+----------+
EmployeeUNI 表:
+----+-----------+
| id | unique_id |
+----+-----------+
| 3  | 1         |
| 11 | 2         |
| 90 | 3         |
+----+-----------+
输出：
+-----------+----------+
| unique_id | name     |
+-----------+----------+
| null      | Alice    |
| null      | Bob      |
| 2         | Meir     |
| 3         | Winston  |
| 1         | Jonathan |
+-----------+----------+
解释：
Alice and Bob 没有唯一标识码, 因此我们使用 null 替代。
Meir 的唯一标识码是 2 。
Winston 的唯一标识码是 3 。
Jonathan 唯一标识码是 1 。
```



1. **LEFT JOIN**操作，将两个表的数据基于 id 列进行组合.使用 LEFT JOIN 来确保将所有 Employees 表中的行都包含在结果中，即使在 EmployeeUNI 表中没有匹配的行。
2. 使用**ON** 链接而不是WHERE



#### **2.SQL解答：**

```sql
SELECT 
	en.unique_id,e.name 
FROM 
	Employees e 
LEFT JOIN 
	EmployeeUNI en
ON 
	e.id=en.id ;
```



#### **3.Pandas解答：**

```Python
import pandas as pd

def replace_employee_id(employees:pd.DataFrame,employee_uni:pd.DataFrame)->pd.DataFrame:
    employee_name_uni=pd.merge(employees,emploee_uni,on='id',how='left')
	answer=employee_name_uni[['unique_id','name']]

```



### [1068. 产品销售分析 I](https://leetcode.cn/problems/product-sales-analysis-i/)

编写解决方案，以获取 `Sales` 表中所有 `sale_id` 对应的 `product_name` 以及该产品的所有 `year` 和 `price` 。

返回结果表 **无顺序要求** 。

结果格式示例如下。

 

#### 1.**示例 ：**

```sql
输入：
Sales 表：
+---------+------------+------+----------+-------+
| sale_id | product_id | year | quantity | price |
+---------+------------+------+----------+-------+ 
| 1       | 100        | 2008 | 10       | 5000  |
| 2       | 100        | 2009 | 12       | 5000  |
| 7       | 200        | 2011 | 15       | 9000  |
+---------+------------+------+----------+-------+
Product 表：
+------------+--------------+
| product_id | product_name |
+------------+--------------+
| 100        | Nokia        |
| 200        | Apple        |
| 300        | Samsung      |
+------------+--------------+
输出：
+--------------+-------+-------+
| product_name | year  | price |
+--------------+-------+-------+
| Nokia        | 2008  | 5000  |
| Nokia        | 2009  | 5000  |
| Apple        | 2011  | 9000  |
+--------------+-------+-------+
```



- 题目【以获取 Sales 表中所有 sale_id 对应的 product_name 】，左连接以Sales表为准



#### **2.SQL解答：**

```sql
SELECT 
  product_name, year,price 
FROM 
  Sales
LEFT JOIN 
  Product
ON Sales.product_id=Product.product_id;
```

#### **3.Pandas解答：**

```python
def sales_analysis(Sales:pd.DataFrame,Product:pd.DataFrame)->pd.DataFrame:
  merged_table=pd.merge(Sales,Product,on='product_id',how='left')
  return merged_table[['product_name','year','price']]
```



### [1581. 进店却未进行过交易的顾客](https://leetcode.cn/problems/customer-who-visited-but-did-not-make-any-transactions/)



有一些顾客可能光顾了购物中心但没有进行交易。请你编写一个解决方案，来查找这些顾客的 ID ，以及他们只光顾不交易的次数。

返回以 **任何顺序** 排序的结果表。

返回结果格式如下例所示。

 

#### 1.**示例：**

```sql
输入:
Visits
+----------+-------------+
| visit_id | customer_id |
+----------+-------------+
| 1        | 23          |
| 2        | 9           |
| 4        | 30          |
| 5        | 54          |
| 6        | 96          |
| 7        | 54          |
| 8        | 54          |
+----------+-------------+
Transactions
+----------------+----------+--------+
| transaction_id | visit_id | amount |
+----------------+----------+--------+
| 2              | 5        | 310    |
| 3              | 5        | 300    |
| 9              | 5        | 200    |
| 12             | 1        | 910    |
| 13             | 2        | 970    |
+----------------+----------+--------+
输出:
+-------------+----------------+
| customer_id | count_no_trans |
+-------------+----------------+
| 54          | 2              |
| 30          | 1              |
| 96          | 1              |
+-------------+----------------+
解释:
ID = 23 的顾客曾经逛过一次购物中心，并在 ID = 12 的访问期间进行了一笔交易。
ID = 9 的顾客曾经逛过一次购物中心，并在 ID = 13 的访问期间进行了一笔交易。
ID = 30 的顾客曾经去过购物中心，并且没有进行任何交易。
ID = 54 的顾客三度造访了购物中心。在 2 次访问中，他们没有进行任何交易，在 1 次访问中，他们进行了 3 次交易。
ID = 96 的顾客曾经去过购物中心，并且没有进行任何交易。
如我们所见，ID 为 30 和 96 的顾客一次没有进行任何交易就去了购物中心。顾客 54 也两次访问了购物中心并且没有进行任何交易。
```



#### **2.SQL解答：**



方法一：

```sql
SELECT 
	v.customer_id, COUNT(v.visit_id) AS count_no_trans
FROM 
	Visits v
WHERE 
	v.visit_id 
NOT IN 
	(SELECT 
     	t.visit_id 
     FROM Transactions t) 
GROUP BY 
	v.customer_id
ORDER BY 
	count_no_trans 
DESC;
```



方法二：

![12.png](media\1601110868-kXhJMt-12.png)



```sql
select 
	V.customer_id, count(distinct V.visit_id) as count_no_trans 
from 
	Visits V 
left join 
	Transactions T
on 
	V.visit_id = T.visit_id 
where 
	T.transaction_id is null
group by
	(V.customer_id)
order by 
	count_no_trans desc
```



### [197. 上升的温度](https://leetcode.cn/problems/rising-temperature/)

编写解决方案，找出与之前（昨天的）日期相比温度更高的所有日期的 `id` 。

返回结果 **无顺序要求** 。

结果格式如下例子所示。

 

#### **1.示例：**

```sql
输入：
Weather 表：
+----+------------+-------------+
| id | recordDate | Temperature |
+----+------------+-------------+
| 1  | 2015-01-01 | 10          |
| 2  | 2015-01-02 | 25          |
| 3  | 2015-01-03 | 20          |
| 4  | 2015-01-04 | 30          |
+----+------------+-------------+
输出：
+----+
| id |
+----+
| 2  |
| 4  |
+----+
解释：
2015-01-02 的温度比前一天高（10 -> 25）
2015-01-04 的温度比前一天高（20 -> 30）
```

- 难度：如何找到前一天日期，如何比较日期数据
  - cross join
  - datediff

https://www.zhihu.com/tardis/zm/art/95768329?source_id=1003



![img](https://pic2.zhimg.com/v2-8100e30a39da4cbb2ad22db5f7e096b9_b.webp?consumer=ZHI_MENG)



#### 2.SQL 解法一：

```sql
SELECT 
	a.id 
FROM 
	Weather a 
CROSS JOIN 
	Weather b 
ON 
	DATEDIFF(a.recordDate,b.recordDate)=1
WHERE 
	a.Temperature>b.Temperature;
```



#### 3. SQL 解法二：

```sql
select 
	w2.id
from 
	Weather w1, Weather w2
where 
	datediff(w2.recordDate, w1.recordDate) = 1 
and 
	w2.Temperature > w1.Temperature
```



### [1661. 每台机器的进程平均运行时间](https://leetcode.cn/problems/average-time-of-process-per-machine/)

现在有一个工厂网站由几台机器运行，每台机器上运行着 **相同数量的进程** 。编写解决方案，计算每台机器各自完成一个进程任务的平均耗时。

完成一个进程任务的时间指进程的`'end' 时间戳` 减去 `'start' 时间戳`。平均耗时通过计算每台机器上所有进程任务的总耗费时间除以机器上的总进程数量获得。

结果表必须包含`machine_id（机器ID）` 和对应的 **average time（平均耗时）** 别名 `processing_time`，且**四舍五入保留3位小数。**

以 **任意顺序** 返回表。

具体参考例子如下。

 

#### 1.**示例 :**

```sql
输入：
Activity table:
+------------+------------+---------------+-----------+
| machine_id | process_id | activity_type | timestamp |
+------------+------------+---------------+-----------+
| 0          | 0          | start         | 0.712     |
| 0          | 0          | end           | 1.520     |
| 0          | 1          | start         | 3.140     |
| 0          | 1          | end           | 4.120     |
| 1          | 0          | start         | 0.550     |
| 1          | 0          | end           | 1.550     |
| 1          | 1          | start         | 0.430     |
| 1          | 1          | end           | 1.420     |
| 2          | 0          | start         | 4.100     |
| 2          | 0          | end           | 4.512     |
| 2          | 1          | start         | 2.500     |
| 2          | 1          | end           | 5.000     |
+------------+------------+---------------+-----------+
输出：
+------------+-----------------+
| machine_id | processing_time |
+------------+-----------------+
| 0          | 0.894           |
| 1          | 0.995           |
| 2          | 1.456           |
+------------+-----------------+
解释：
一共有3台机器,每台机器运行着两个进程.
机器 0 的平均耗时: ((1.520 - 0.712) + (4.120 - 3.140)) / 2 = 0.894
机器 1 的平均耗时: ((1.550 - 0.550) + (1.420 - 0.430)) / 2 = 0.995
机器 2 的平均耗时: ((4.512 - 4.100) + (5.000 - 2.500)) / 2 = 1.456
```

难点：

- AVG 的使用
- ROUND() 以及小数点的使用
- 一个表实现

#### 2. SQL 两个表链接：

```sql

SELECT 
	a.machine_id,ROUND(AVG(b.timestamp-a.timestamp),3) AS processing_time
FROM 
	Activity a
INNER JOIN 
	Activity b 
ON 
	a.machine_id=b.machine_id 
AND 
	a.process_id=b.process_id 
AND 
	a.activity_type='start' 
AND 
	b.activity_type='end' 
GROUP BY a.machine_id;
```

#### 3. SQL 一个表实现：

如果start减时间，否则加时间，算一次时间差共进行两次，在计算avg时分母是原来两倍，因此分子要乘2

```sql

SELECT machine_id AS 'machine_id',
    ROUND(
        SUM(IF(activity_type = 'start', -timestamp, timestamp))
        / COUNT(*) 
        * 2
        ,3) AS 'processing_time'
FROM Activity
GROUP BY machine_id;

```



## Day 3

### [577. 员工奖金](https://leetcode.cn/problems/employee-bonus/)

编写解决方案，报告每个奖金 **少于** `1000` 的员工的姓名和奖金数额。

以 **任意顺序** 返回结果表。

结果格式如下所示。

 

#### **1.示例：**

```sql
输入：
Employee table:
+-------+--------+------------+--------+
| empId | name   | supervisor | salary |
+-------+--------+------------+--------+
| 3     | Brad   | null       | 4000   |
| 1     | John   | 3          | 1000   |
| 2     | Dan    | 3          | 2000   |
| 4     | Thomas | 3          | 4000   |
+-------+--------+------------+--------+
Bonus table:
+-------+-------+
| empId | bonus |
+-------+-------+
| 2     | 500   |
| 4     | 2000  |
+-------+-------+
输出：
+------+-------+
| name | bonus |
+------+-------+
| Brad | null  |
| John | null  |
| Dan  | 500   |
+------+-------+
```



#### 2.SQL 解法：

```sql
SELECT 
  e.name, 
  b.bonus 
FROM 
  Employee e 
  LEFT JOIN Bonus b ON e.empId = b.empId 
WHERE 
  b.bonus < 1000 
  OR b.bonus IS NULL;
```



### [1280. 学生们参加各科测试的次数](https://leetcode.cn/problems/students-and-examinations/)



查询出每个学生参加每一门科目测试的次数，结果按 `student_id` 和 `subject_name` 排序。

查询结构格式如下所示。

 

#### **1.示例:**

```sql
输入：
Students table:
+------------+--------------+
| student_id | student_name |
+------------+--------------+
| 1          | Alice        |
| 2          | Bob          |
| 13         | John         |
| 6          | Alex         |
+------------+--------------+
Subjects table:
+--------------+
| subject_name |
+--------------+
| Math         |
| Physics      |
| Programming  |
+--------------+
Examinations table:
+------------+--------------+
| student_id | subject_name |
+------------+--------------+
| 1          | Math         |
| 1          | Physics      |
| 1          | Programming  |
| 2          | Programming  |
| 1          | Physics      |
| 1          | Math         |
| 13         | Math         |
| 13         | Programming  |
| 13         | Physics      |
| 2          | Math         |
| 1          | Math         |
+------------+--------------+
输出：
+------------+--------------+--------------+----------------+
| student_id | student_name | subject_name | attended_exams |
+------------+--------------+--------------+----------------+
| 1          | Alice        | Math         | 3              |
| 1          | Alice        | Physics      | 2              |
| 1          | Alice        | Programming  | 1              |
| 2          | Bob          | Math         | 1              |
| 2          | Bob          | Physics      | 0              |
| 2          | Bob          | Programming  | 1              |
| 6          | Alex         | Math         | 0              |
| 6          | Alex         | Physics      | 0              |
| 6          | Alex         | Programming  | 0              |
| 13         | John         | Math         | 1              |
| 13         | John         | Physics      | 1              |
| 13         | John         | Programming  | 1              |
+------------+--------------+--------------+----------------+
解释：
结果表需包含所有学生和所有科目（即便测试次数为0）：
Alice 参加了 3 次数学测试, 2 次物理测试，以及 1 次编程测试；
Bob 参加了 1 次数学测试, 1 次编程测试，没有参加物理测试；
Alex 啥测试都没参加；
John  参加了数学、物理、编程测试各 1 次。
```





#### 2.思路

- 结果的前三列列为表students和subjects的交叉连接，也就是笛卡尔积 
- 而最后一列为每个学生参加每个学科的测试次数，也就是分组统计

1.先连结两个表：对表Students和表Subjects求笛卡尔积

```sql
SELECT 
  * 
FROM 
  Students as a 
  CROSS JOIN Subjects as b 
ORDER BY 
  a.student_id, 
  b.subject_name;
```

2.分组统计

```sql
select
student_id, 
subject_name,
count(student_id) as attended_exams
from `Examinations`
group by student_id,subject_name
```

3. 再对刚刚得到的新表和表Examinations进行左连结（LEFT JOIN）

```sql
SELECT 
  a.student_id, 
  a.student_name, 
  b.subject_name, 
  COUNT(e.student_id) AS attended_exams 
FROM 
  Students as a 
  CROSS JOIN Subjects as b 
  LEFT JOIN Examinations e ON a.student_id = e.student_id 
  AND b.subject_name = e.subject_name 
GROUP BY 
  a.student_id, 
  b.subject_name 
ORDER BY 
  a.student_id, 
  b.subject_name;
```



### [570. 至少有5名直接下属的经理](https://leetcode.cn/problems/managers-with-at-least-5-direct-reports/)

查询**至少有5名直接下属**的经理 。

以 **任意顺序** 返回结果表。

查询结果格式如下所示。

**示例 1:**

```sql
输入: 
Employee 表:
+-----+-------+------------+-----------+
| id  | name  | department | managerId |
+-----+-------+------------+-----------+
| 101 | John  | A          | None      |
| 102 | Dan   | A          | 101       |
| 103 | James | A          | 101       |
| 104 | Amy   | A          | 101       |
| 105 | Anne  | A          | 101       |
| 106 | Ron   | B          | 101       |
+-----+-------+------------+-----------+
输出: 
+------+
| name |
+------+
| John |
+------+
```



- HAVING 的用法
- 自连结

```sql
SELECT 
  b.name 
FROM 
  Employee a 
  INNER JOIN Employee b ON a.managerId = b.id 
GROUP BY 
  a.managerId 
HAVING 
  COUNT(a.managerId)>= 5;

```

- in + 子查询

```sql
SELECT name
FROM Employee
where id in
(
SELECT ManagerID
FROM Employee 
GROUP BY ManagerID
HAVING count(ManagerID)>=5
)
```



### [1934. 确认率](https://leetcode.cn/problems/confirmation-rate/)

用户的 **确认率** 是 `'confirmed'` 消息的数量除以请求的确认消息的总数。没有请求任何确认消息的用户的确认率为 `0` 。确认率四舍五入到 **小数点后两位** 。

编写一个SQL查询来查找每个用户的 确认率 。

以 任意顺序 返回结果表。

查询结果格式如下所示。

#### **1.示例:**

```sql
输入：
Signups 表:
+---------+---------------------+
| user_id | time_stamp          |
+---------+---------------------+
| 3       | 2020-03-21 10:16:13 |
| 7       | 2020-01-04 13:57:59 |
| 2       | 2020-07-29 23:09:44 |
| 6       | 2020-12-09 10:39:37 |
+---------+---------------------+
Confirmations 表:
+---------+---------------------+-----------+
| user_id | time_stamp          | action    |
+---------+---------------------+-----------+
| 3       | 2021-01-06 03:30:46 | timeout   |
| 3       | 2021-07-14 14:00:00 | timeout   |
| 7       | 2021-06-12 11:57:29 | confirmed |
| 7       | 2021-06-13 12:58:28 | confirmed |
| 7       | 2021-06-14 13:59:27 | confirmed |
| 2       | 2021-01-22 00:00:00 | confirmed |
| 2       | 2021-02-28 23:59:59 | timeout   |
+---------+---------------------+-----------+
输出: 
+---------+-------------------+
| user_id | confirmation_rate |
+---------+-------------------+
| 6       | 0.00              |
| 3       | 0.00              |
| 7       | 1.00              |
| 2       | 0.50              |
+---------+-------------------+
解释:
用户 6 没有请求任何确认消息。确认率为 0。
用户 3 进行了 2 次请求，都超时了。确认率为 0。
用户 7 提出了 3 个请求，所有请求都得到了确认。确认率为 1。
用户 2 做了 2 个请求，其中一个被确认，另一个超时。确认率为 1 / 2 = 0.5。
```



#### 2.SQL 解法：

```sql
SELECT 
  s.user_id, 
  ROUND(
    AVG(
      IF(c.action = 'confirmed', 1, 0)
    ), 
    2
  ) AS confirmation_rate 
FROM 
  Signups s 
  LEFT JOIN Confirmations c ON s.user_id = c.user_id 
GROUP BY 
  s.user_id;
```



- IFNULL 的用法

```sql
ROUND(IFNULL(AVG(c.action='confirmed'), 0), 2) AS confirmation_rate
```



### [620. 有趣的电影](https://leetcode.cn/problems/not-boring-movies/)

编写解决方案，找出所有影片描述为 **非** `boring` (不无聊) 的并且 **id 为奇数** 的影片。

返回结果按 `rating` **降序排列**。

结果格式如下示例。

 

#### **1.示例：**

```sql
输入：
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   1     | War       |   great 3D   |   8.9     |
|   2     | Science   |   fiction    |   8.5     |
|   3     | irish     |   boring     |   6.2     |
|   4     | Ice song  |   Fantacy    |   8.6     |
|   5     | House card|   Interesting|   9.1     |
+---------+-----------+--------------+-----------+
输出：
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   5     | House card|   Interesting|   9.1     |
|   1     | War       |   great 3D   |   8.9     |
+---------+-----------+--------------+-----------+
解释：
我们有三部电影，它们的 id 是奇数:1、3 和 5。id = 3 的电影是 boring 的，所以我们不把它包括在答案中。
```



#### 2.SQL 解法：

难点：

- 字符串的匹配
- 查询奇数
  - mod(id,2)=1
  - id % 2 = 1
  - `x & 1 = 1` ，如果是 `1` 就是奇数

```sql
SELECT 
  * 
FROM 
  cinema 
WHERE 
  description <> 'boring' 
  AND id % 2 = 1 
ORDER BY 
  rating DESC;
```

