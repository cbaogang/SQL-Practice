高频 SQL 50 题（基础版）

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
JOIN 
	Employee b 
ON 
	a.managerId = b.id 
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





## Day 4

### [**1251. 平均售价**](https://leetcode.cn/problems/average-selling-price/)



编写解决方案以查找每种产品的平均售价。`average_price` 应该 **四舍五入到小数点后两位**。

返回结果表 **无顺序要求** 。

结果格式如下例所示。

 

**示例 1：**

```sql
输入：
Prices table:
+------------+------------+------------+--------+
| product_id | start_date | end_date   | price  |
+------------+------------+------------+--------+
| 1          | 2019-02-17 | 2019-02-28 | 5      |
| 1          | 2019-03-01 | 2019-03-22 | 20     |
| 2          | 2019-02-01 | 2019-02-20 | 15     |
| 2          | 2019-02-21 | 2019-03-31 | 30     |
+------------+------------+------------+--------+
UnitsSold table:
+------------+---------------+-------+
| product_id | purchase_date | units |
+------------+---------------+-------+
| 1          | 2019-02-25    | 100   |
| 1          | 2019-03-01    | 15    |
| 2          | 2019-02-10    | 200   |
| 2          | 2019-03-22    | 30    |
+------------+---------------+-------+
输出：
+------------+---------------+
| product_id | average_price |
+------------+---------------+
| 1          | 6.96          |
| 2          | 16.96         |
+------------+---------------+
解释：
平均售价 = 产品总价 / 销售的产品数量。
产品 1 的平均售价 = ((100 * 5)+(15 * 20) )/ 115 = 6.96
产品 2 的平均售价 = ((200 * 15)+(30 * 30) )/ 230 = 16.96
```



注意点：

- 考虑售卖为0 的情况，也就是Prices table 有值，而UnitsSold table 无值的情况

#### SQL 解答：

```sql
SELECT 
  a.product_id, 
  IFNULL(
    ROUND(
      SUM(b.units * a.price)/ SUM(b.units), 
      2
    ), 
    0
  ) AS average_price 
FROM 
  Prices AS a 
  LEFT JOIN UnitsSold AS b ON a.product_id = b.product_id 
  AND b.purchase_date BETWEEN a.start_date 
  AND a.end_date 
GROUP BY 
  a.product_id;
```



### [1075. 项目员工 I](https://leetcode.cn/problems/project-employees-i/)



请写一个 SQL 语句，查询每一个项目中员工的 **平均** 工作年限，**精确到小数点后两位**。

查询结果的格式如下：

```sql
Project 表：
+-------------+-------------+
| project_id  | employee_id |
+-------------+-------------+
| 1           | 1           |
| 1           | 2           |
| 1           | 3           |
| 2           | 1           |
| 2           | 4           |
+-------------+-------------+

Employee 表：
+-------------+--------+------------------+
| employee_id | name   | experience_years |
+-------------+--------+------------------+
| 1           | Khaled | 3                |
| 2           | Ali    | 2                |
| 3           | John   | 1                |
| 4           | Doe    | 2                |
+-------------+--------+------------------+

Result 表：
+-------------+---------------+
| project_id  | average_years |
+-------------+---------------+
| 1           | 2.00          |
| 2           | 2.50          |
+-------------+---------------+
第一个项目中，员工的平均工作年限是 (3 + 2 + 1) / 3 = 2.00；第二个项目中，员工的平均工作年限是 (3 + 2) / 2 = 2.50
```



#### 2.SQL 解答：

```sql
SELECT 
  a.project_id, 
  ROUND(
    AVG(b.experience_years), 
    2
  ) AS average_years 
FROM 
  Project AS a 
  LEFT JOIN Employee AS b ON a.employee_id = b.employee_id 
GROUP BY 
  a.project_id 
ORDER BY 
  a.employee_id;
```



### [1633. 各赛事的用户注册率](https://leetcode.cn/problems/percentage-of-users-attended-a-contest/)

编写解决方案统计出各赛事的用户注册百分率，保留两位小数。

返回的结果表按 `percentage` 的 **降序** 排序，若相同则按 `contest_id` 的 **升序** 排序。

返回结果如下示例所示。

 

**示例 1：**

```sql
输入：
Users 表：
+---------+-----------+
| user_id | user_name |
+---------+-----------+
| 6       | Alice     |
| 2       | Bob       |
| 7       | Alex      |
+---------+-----------+

Register 表：
+------------+---------+
| contest_id | user_id |
+------------+---------+
| 215        | 6       |
| 209        | 2       |
| 208        | 2       |
| 210        | 6       |
| 208        | 6       |
| 209        | 7       |
| 209        | 6       |
| 215        | 7       |
| 208        | 7       |
| 210        | 2       |
| 207        | 2       |
| 210        | 7       |
+------------+---------+
输出：
+------------+------------+
| contest_id | percentage |
+------------+------------+
| 208        | 100.0      |
| 209        | 100.0      |
| 210        | 100.0      |
| 215        | 66.67      |
| 207        | 33.33      |
+------------+------------+
解释：
所有用户都注册了 208、209 和 210 赛事，因此这些赛事的注册率为 100% ，我们按 contest_id 的降序排序加入结果表中。
Alice 和 Alex 注册了 215 赛事，注册率为 ((2/3) * 100) = 66.67%
Bob 注册了 207 赛事，注册率为 ((1/3) * 100) = 33.33%
```





那么要找准分子和分母是什么。

- 分子： Register表中每个contest对应的user_id个数

- 分母： 分母就是3，因为user表里一共就3个人，当然准确来说是user表里面的所有人

注意：

- 100要放在round里面
- count(1)和count(*)都差不多，没啥区别
- 分子的count(user_id)不需要加distinct，因为(contest_id, user_id) 是联合主键，在group by contest_id之后，user_id必然是不会重复的



#### 2.SQL解答：

```sql
SELECT 
  b.contest_id, 
  ROUND(
    COUNT(b.user_id)/(
      SELECT 
        COUNT(*) 
      FROM 
        Users
    )* 100, 
    2
  ) AS percentage 
FROM 
  Register AS b 
GROUP BY 
  b.contest_id 
ORDER BY 
  percentage DESC, 
  b.contest_id ASC;
```



### [1211. 查询结果的质量和占比](https://leetcode.cn/problems/queries-quality-and-percentage/)



将查询结果的质量 `quality` 定义为：

> 各查询结果的评分与其位置之间比率的平均值。

将劣质查询百分比 `poor_query_percentage` 为：

> 评分小于 3 的查询结果占全部查询结果的百分比。

编写解决方案，找出每次的 `query_name` 、 `quality` 和 `poor_query_percentage`。

`quality` 和 `poor_query_percentage` 都应 **四舍五入到小数点后两位** 。

以 **任意顺序** 返回结果表。

结果格式如下所示：

 

**示例 1：**

```sql
输入：
Queries table:
+------------+-------------------+----------+--------+
| query_name | result            | position | rating |
+------------+-------------------+----------+--------+
| Dog        | Golden Retriever  | 1        | 5      |
| Dog        | German Shepherd   | 2        | 5      |
| Dog        | Mule              | 200      | 1      |
| Cat        | Shirazi           | 5        | 2      |
| Cat        | Siamese           | 3        | 3      |
| Cat        | Sphynx            | 7        | 4      |
+------------+-------------------+----------+--------+
输出：
+------------+---------+-----------------------+
| query_name | quality | poor_query_percentage |
+------------+---------+-----------------------+
| Dog        | 2.50    | 33.33                 |
| Cat        | 0.66    | 33.33                 |
+------------+---------+-----------------------+
解释：
Dog 查询结果的质量为 ((5 / 1) + (5 / 2) + (1 / 200)) / 3 = 2.50
Dog 查询结果的劣质查询百分比为 (1 / 3) * 100 = 33.33

Cat 查询结果的质量为 ((2 / 5) + (3 / 3) + (4 / 7)) / 3 = 0.66
Cat 查询结果的劣质查询百分比为 (1 / 3) * 100 = 33.33
```



#### 2.SQL 解法：

- 使用IF(rating<3,1,0) 来筛选

```sql
SELECT 
  query_name, 
  ROUND(AVG(rating / position), 2) AS quality, 
  ROUND(AVG(IF(rating < 3, 1, 0))* 100, 2) AS poor_query_percentage 
FROM 
  Queries 
GROUP BY 
  query_name;
```



- 直接使用AVG(rating<3)来筛选

```sql
select
    query_name,
    round(avg(rating/position),2) quality,
    round(100 * avg(rating < 3),2) poor_query_percentage
from
    Queries
group by
    query_name;

```



### [1193. 每月交易 I](https://leetcode.cn/problems/monthly-transactions-i/)

编写一个 sql 查询来查找每个月和每个国家/地区的事务数及其总金额、已批准的事务数及其总金额。

以 **任意顺序** 返回结果表。

查询结果格式如下所示。

 

**示例 1:**

```sql
输入：
Transactions table:
+------+---------+----------+--------+------------+
| id   | country | state    | amount | trans_date |
+------+---------+----------+--------+------------+
| 121  | US      | approved | 1000   | 2018-12-18 |
| 122  | US      | declined | 2000   | 2018-12-19 |
| 123  | US      | approved | 2000   | 2019-01-01 |
| 124  | DE      | approved | 2000   | 2019-01-07 |
+------+---------+----------+--------+------------+
输出：
+----------+---------+-------------+----------------+--------------------+-----------------------+
| month    | country | trans_count | approved_count | trans_total_amount | approved_total_amount |
+----------+---------+-------------+----------------+--------------------+-----------------------+
| 2018-12  | US      | 2           | 1              | 3000               | 1000                  |
| 2019-01  | US      | 1           | 1              | 2000               | 2000                  |
| 2019-01  | DE      | 1           | 1              | 2000               | 2000                  |
+----------+---------+-------------+----------------+--------------------+-----------------------+
```



难点：

- 1.使用DATE_FORMAT()函数把日期转换成月份 ：

  | %Y   | Year as a numeric, 4-digit value                             |
  | ---- | ------------------------------------------------------------ |
  | %y   | Year as a numeric, 2-digit value                             |
  | %M   | Month name in full (January to December)                     |
  | %m   | Month name as a numeric value (00 to 12)                     |
  | %D   | Day of the month as a numeric value, followed by suffix (1st, 2nd, 3rd, ...) |
  | %d   | Day of the month as a numeric value (01 to 31)               |

- 2.count(),sum()函数在聚合时加入条件 （sum()函数中返回0,count()函数中返回null可以过滤掉不符合记录）

- 因为count统计原理：只要有值不是null都会计数



#### 2.SQL 解答：

```sql
SELECT 
  DATE_FORMAT(trans_date, "%Y-%m") AS month, 
  country, 
  COUNT(*) AS trans_count, 
  COUNT(IF(state = 'approved', 1, NULL)) AS approved_count, 
  SUM(amount) AS trans_total_amount, 
  SUM(IF(state = 'approved', amount, 0)) AS approved_total_amount 
FROM 
  Transactions 
GROUP BY 
  month, 
  country;
```



## Day 5

### [1174. 即时食物配送 II](https://leetcode.cn/problems/immediate-food-delivery-ii/)

如果顾客期望的配送日期和下单日期相同，则该订单称为 「**即时订单**」，否则称为「**计划订单**」。

「**首次订单**」是顾客最早创建的订单。我们保证一个顾客只会有一个「首次订单」。

编写解决方案以获取即时订单在所有用户的首次订单中的比例。**保留两位小数。**

结果示例如下所示：

 

#### **1.示例：**

```sql
输入：
Delivery 表：
+-------------+-------------+------------+-----------------------------+
| delivery_id | customer_id | order_date | customer_pref_delivery_date |
+-------------+-------------+------------+-----------------------------+
| 1           | 1           | 2019-08-01 | 2019-08-02                  |
| 2           | 2           | 2019-08-02 | 2019-08-02                  |
| 3           | 1           | 2019-08-11 | 2019-08-12                  |
| 4           | 3           | 2019-08-24 | 2019-08-24                  |
| 5           | 3           | 2019-08-21 | 2019-08-22                  |
| 6           | 2           | 2019-08-11 | 2019-08-13                  |
| 7           | 4           | 2019-08-09 | 2019-08-09                  |
+-------------+-------------+------------+-----------------------------+
输出：
+----------------------+
| immediate_percentage |
+----------------------+
| 50.00                |
+----------------------+
解释：
1 号顾客的 1 号订单是首次订单，并且是计划订单。
2 号顾客的 2 号订单是首次订单，并且是即时订单。
3 号顾客的 5 号订单是首次订单，并且是计划订单。
4 号顾客的 7 号订单是首次订单，并且是即时订单。
因此，一半顾客的首次订单是即时的。
```





#### 2.解题思路


- 先将每个消费者的第一次消费日期找出来

  - MIN 函数的使用
  - GROUP BY 

```sql
SELECT customer_id, MIN(order_date) AS order_date
FROM Delivery 
GROUP BY customer_id;
```



- 再将临时表扩展customer_pref_delivery_date字段,当满足order_date = customer_pref_delivery_date时进行计数除以总人数

```sql
SELECT 
  ROUND(SUM(order_date=customer_pref_delivery_date)*100/COUNT(*),2) 
AS immediate_percentage 
FROM Delivery 
WHERE (customer_id ,order_date ) in 
	(SELECT customer_id,MIN(order_date) AS order_date 
     FROM Delivery 
     GROUP BY customer_id 
    );
```



```sql
SELECT 
	ROUND(AVG(order_date = customer_pref_delivery_date)*100,2) AS immediate_percentage 
FROM 
	Delivery 
WHERE 
	(customer_id,order_date) 
IN 
	(SELECT 
         customer_id,min(order_date) AS order_date 
     FROM 
         Delivery 
     GROUP BY 
         customer_id);
```



### [550. 游戏玩法分析 IV](https://leetcode.cn/problems/game-play-analysis-iv/)

编写解决方案，报告在首次登录的第二天再次登录的玩家的 **比率**，**四舍五入到小数点后两位**。换句话说，你需要计算从**首次登录日期开始至少连续两天登录的玩家的数量，然后除以玩家总数**。

结果格式如下所示：

 

**示例 1：**

```sql
输入：
Activity table:
+-----------+-----------+------------+--------------+
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-03-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |
+-----------+-----------+------------+--------------+
输出：
+-----------+
| fraction  |
+-----------+
| 0.33      |
+-----------+
解释：
只有 ID 为 1 的玩家在第一天登录后才重新登录，所以答案是 1/3 = 0.33
```





### 解题思路

先过滤出每个用户的首次登陆日期，然后左关联，筛选次日存在的记录的比例

- **首次登录日期开始至少连续两天登录的玩家的数量，然后除以玩家总数**。

1. 首次登陆的玩家

```sql
select player_id, min(event_date) as event_date
from activity
group by player_id
```

2. 首次登录的第二天登录

```sql
DATEDIFF(a.event,b.event_date)=1
```



```sql
select round(avg(a.event_date is not null), 2) fraction
from 
    (select player_id, min(event_date) as login
    from activity
    group by player_id) p 
left join activity a 
on p.player_id=a.player_id and datediff(a.event_date, p.login)=1

```

其他方法：

```sql
select ROUND(COUNT(b.event_date)/count(*),2) AS fraction 
from 
	(SELECT player_id,min(event_date) AS event_date 
     FROM Activity 
     GROUP BY player_id) a 
LEFT JOIN Activity b 
ON a.player_id=b.player_id and datediff(b.event_date,a.event_date)=1;
```



```sql
WITH PlayerFirstLogin AS (
    SELECT
        player_id,
        MIN(event_date) AS first_login_date
    FROM
        Activity
    GROUP BY
        player_id
)

SELECT
    ROUND(SUM(CASE WHEN DATEDIFF(a.event_date, pfl.first_login_date) = 1 THEN 1 ELSE 0 END) / COUNT(DISTINCT a.player_id), 2) AS fraction
FROM
    Activity a
JOIN
    PlayerFirstLogin pfl ON a.player_id = pfl.player_id;

```



### [2356. 每位教师所教授的科目种类的数量](https://leetcode.cn/problems/number-of-unique-subjects-taught-by-each-teacher/)



查询每位老师在大学里教授的**科目种类**的数量。

以 **任意顺序** 返回结果表。

查询结果格式示例如下。

 

**示例 1:**

```sql
输入: 
Teacher 表:
+------------+------------+---------+
| teacher_id | subject_id | dept_id |
+------------+------------+---------+
| 1          | 2          | 3       |
| 1          | 2          | 4       |
| 1          | 3          | 3       |
| 2          | 1          | 1       |
| 2          | 2          | 1       |
| 2          | 3          | 1       |
| 2          | 4          | 1       |
+------------+------------+---------+
输出:  
+------------+-----+
| teacher_id | cnt |
+------------+-----+
| 1          | 2   |
| 2          | 4   |
+------------+-----+
解释: 
教师 1:
  - 他在 3、4 系教科目 2。
  - 他在 3 系教科目 3。
教师 2:
  - 他在 1 系教科目 1。
  - 他在 1 系教科目 2。
  - 他在 1 系教科目 3。
  - 他在 1 系教科目 4。
```



```sql
SELECT teacher_id ,COUNT(DISTINCT subject_id) AS cnt
FROM Teacher 
GROUP BY 1; 
```



#### Pandas:

解题思路

- 分组和聚合：unique_subjects = teacher.groupby('teacher_id')['subject_id'].nunique().reset_index() 这行代码使用 groupby 函数按照 'teacher_id' 列对数据进行分组，然后使用 nunique 函数获取每个分组中不重复的 'subject_id' 的数量。 这意味着在每个教师的分组中，我们只计算不重复的科目数。最后，使用 reset_index 方法重置索引，生成一个新的 DataFrame。
- 重命名列名：unique_subjects = unique_subjects.rename(columns={'subject_id': 'cnt'}) 这行代码使用 rename 函数将 DataFrame 的列名 'subject_id' 改为 'cnt'，以更好地表示每位教师所教授的科目种类数量。



```python
import pandas as pd

def count_unique_subjects(teacher: pd.DataFrame) -> pd.DataFrame:
    unique_subjects = teacher.groupby('teacher_id')['subject_id'].nunique().reset_index()
    unique_subjects = unique_subjects.rename(columns={'subject_id': 'cnt'})
    return unique_subjects

```



### [1141. 查询近30天活跃用户数](https://leetcode.cn/problems/user-activity-for-the-past-30-days-i/)



编写解决方案，统计截至 `2019-07-27`（包含2019-07-27），近 `30` 天的每日活跃用户数（当天只要有一条活动记录，即为活跃用户）。

以 **任意顺序** 返回结果表。

结果示例如下。

 

**示例 1:**

```sql
输入：
Activity table:
+---------+------------+---------------+---------------+
| user_id | session_id | activity_date | activity_type |
+---------+------------+---------------+---------------+
| 1       | 1          | 2019-07-20    | open_session  |
| 1       | 1          | 2019-07-20    | scroll_down   |
| 1       | 1          | 2019-07-20    | end_session   |
| 2       | 4          | 2019-07-20    | open_session  |
| 2       | 4          | 2019-07-21    | send_message  |
| 2       | 4          | 2019-07-21    | end_session   |
| 3       | 2          | 2019-07-21    | open_session  |
| 3       | 2          | 2019-07-21    | send_message  |
| 3       | 2          | 2019-07-21    | end_session   |
| 4       | 3          | 2019-06-25    | open_session  |
| 4       | 3          | 2019-06-25    | end_session   |
+---------+------------+---------------+---------------+
输出：
+------------+--------------+ 
| day        | active_users |
+------------+--------------+ 
| 2019-07-20 | 2            |
| 2019-07-21 | 2            |
+------------+--------------+ 
解释：注意非活跃用户的记录不需要展示。
```



注意点：

- datediff还要限制>=0不然会查询到07-27之后的数据

- 在编写用户类型的题目时，注意观察题目给出的表是否设计主键，是否存在重复，记得加上distinct
- 日期在比较时，需要在时间上加上引号

```sql
SELECT activity_date AS day, COUNT(DISTINCT user_id) AS active_users 
FROM Activity 
WHERE DATEDIfF("2019-07-27",activity_date) <30 AND DATEDIfF("2019-07-27",activity_date)>=0 
GROUP BY activity_date;
```





### [1084. 销售分析III](https://leetcode.cn/problems/sales-analysis-iii/)

编写解决方案，报告`2019年春季`才售出的产品。即**仅**在`**2019-01-01**`至`**2019-03-31**`（含）之间出售的商品。

以 **任意顺序** 返回结果表。

结果格式如下所示。

 

**示例 1:**

```sql
输入：
Product table:
+------------+--------------+------------+
| product_id | product_name | unit_price |
+------------+--------------+------------+
| 1          | S8           | 1000       |
| 2          | G4           | 800        |
| 3          | iPhone       | 1400       |
+------------+--------------+------------+
Sales table:
+-----------+------------+----------+------------+----------+-------+
| seller_id | product_id | buyer_id | sale_date  | quantity | price |
+-----------+------------+----------+------------+----------+-------+
| 1         | 1          | 1        | 2019-01-21 | 2        | 2000  |
| 1         | 2          | 2        | 2019-02-17 | 1        | 800   |
| 2         | 2          | 3        | 2019-06-02 | 1        | 800   |
| 3         | 3          | 4        | 2019-05-13 | 2        | 2800  |
+-----------+------------+----------+------------+----------+-------+
输出：
+-------------+--------------+
| product_id  | product_name |
+-------------+--------------+
| 1           | S8           |
+-------------+--------------+
解释:
id为1的产品仅在2019年春季销售。
id为2的产品在2019年春季销售，但也在2019年春季之后销售。
id 3的产品在2019年春季之后销售。
我们只退回产品1，因为它是2019年春季才销售的产品。
```



难点：

- 使用having
- MIN(), MAX()函数的使用



不用HAVING,用where

```sql
select s.product_id, p.product_name
from Sales s
JOIN Product p
ON s.product_id=p.product_id
where s.product_id not in (
    select product_id
    from sales
    where sale_date >= '2019-04-01' or sale_date < '2019-01-01')
GROUP BY s.product_id;
```



使用HAVING

```sql
select sales.product_id as product_id, product.product_name as product_name
from sales left join product on sales.product_id = product.product_id
group by product_id
having count(sale_date between '2019-01-01' and '2019-03-31' or null) = count(*)

```



```sql
HAVING MIN(sale_date) >= '2019-01-01' AND MAX(sale_date) <= '2019-03-31'
```



## Day 6

### [619. 只出现一次的最大数字](https://leetcode.cn/problems/biggest-single-number/)



**单一数字** 是在 `MyNumbers` 表中只出现一次的数字。

找出最大的 **单一数字** 。如果不存在 **单一数字** ，则返回 `null` 。

查询结果如下例所示。

#### **1.示例 ：**

```
输入：
MyNumbers 表：
+-----+
| num |
+-----+
| 8   |
| 8   |
| 3   |
| 3   |
| 1   |
| 4   |
| 5   |
| 6   |
+-----+
输出：
+-----+
| num |
+-----+
| 6   |
+-----+
解释：单一数字有 1、4、5 和 6 。
6 是最大的单一数字，返回 6 。
```

**示例 2：**

```SQL
输入：
MyNumbers table:
+-----+
| num |
+-----+
| 8   |
| 8   |
| 7   |
| 7   |
| 3   |
| 3   |
| 3   |
+-----+
输出：
+------+
| num  |
+------+
| null |
+------+
解释：输入的表中不存在单一数字，所以返回 null 。
```



难点：

- 如何找出只出现一次的数字

- 空值时如何返回NULL

  - 使用聚合函数进行空值null值的转换: 只要我们的表格为空，加入任何SUM/AVG/MAX/MIN函数，我们都可以得到null值的结果。

  - ifnull函数定位：用于判断第一个表达式是否为 NULL，如果为 NULL 则返回第二个参数的值，如果不为 NULL 则返回第一个参数的值。

  - select 空 param1 -> param1：null

  - select param1 from 空 —> param1：空

  - 可以使用select语句进行转换，但空值应直接写在select中而非from中

    

```sql
select max(a.num) AS num FROM (
SELECT num FROM MyNumbers 
GROUP BY num
HAVING COUNT(*)=1) a;
```





### [1045. 买下所有产品的客户](https://leetcode.cn/problems/customers-who-bought-all-products/)

编写解决方案，报告 `Customer` 表中购买了 `Product` 表中所有产品的客户的 id。

返回结果表 **无顺序要求** 。

返回结果格式如下所示。

 

**示例 1：**

```sql
输入：
Customer 表：
+-------------+-------------+
| customer_id | product_key |
+-------------+-------------+
| 1           | 5           |
| 2           | 6           |
| 3           | 5           |
| 3           | 6           |
| 1           | 6           |
+-------------+-------------+
Product 表：
+-------------+
| product_key |
+-------------+
| 5           |
| 6           |
+-------------+
输出：
+-------------+
| customer_id |
+-------------+
| 1           |
| 3           |
+-------------+
解释：
购买了所有产品（5 和 6）的客户的 id 是 1 和 3 。
```



```sql
SELECT 
	customer_id 
FROM  
	Customer 
GROUP BY 
	customer_id 
HAVING COUNT(DISTINCT product_key) = (SELECT COUNT(*) from Product);
```



### [1731. 每位经理的下属员工数量](https://leetcode.cn/problems/the-number-of-employees-which-report-to-each-employee/)



将至少有一个其他员工需要向他汇报的员工，视为一个经理。

编写SQL查询需要听取汇报的所有经理的ID、名称、直接向该经理汇报的员工人数，以及这些员工的平均年龄，其中该平均年龄需要四舍五入到最接近的整数。

返回的结果集需要按照 `employee_id `进行排序。

查询结果的格式如下：

 

**注意点：**

- 使用JOIN 而不是left 或者right JOIN, 直接剔除了NULL 值

```sql
Employees table:
+-------------+---------+------------+-----+
| employee_id | name    | reports_to | age |
+-------------+---------+------------+-----+
| 9           | Hercy   | null       | 43  |
| 6           | Alice   | 9          | 41  |
| 4           | Bob     | 9          | 36  |
| 2           | Winston | null       | 37  |
+-------------+---------+------------+-----+

Result table:
+-------------+-------+---------------+-------------+
| employee_id | name  | reports_count | average_age |
+-------------+-------+---------------+-------------+
| 9           | Hercy | 2             | 39          |
+-------------+-------+---------------+-------------+
Hercy 有两个需要向他汇报的员工, 他们是 Alice and Bob. 他们的平均年龄是 (41+36)/2 = 38.5, 四舍五入的结果是 39.
```



```sql
SELECT 
	b.employee_id,b.name,COUNT(a.name) as reports_count,round(AVG(a.age),0) as average_age
FROM 
	Employees a
JOIN 
	Employees b 
ON 
	a.reports_to=b.employee_id 
GROUP BY b.employee_id 
ORDER BY 1;
```



## Day 7

### [1789. 员工的直属部门](https://leetcode.cn/problems/primary-department-for-each-employee/)

一个员工可以属于多个部门。当一个员工加入**超过一个部门**的时候，他需要决定哪个部门是他的直属部门。请注意，当员工只加入一个部门的时候，那这个部门将默认为他的直属部门，虽然表记录的值为`'N'`.

请编写解决方案，查出员工所属的直属部门。

返回结果 **没有顺序要求** 。

返回结果格式如下例子所示：

 

**示例 1：**

```sql
输入：
Employee table:
+-------------+---------------+--------------+
| employee_id | department_id | primary_flag |
+-------------+---------------+--------------+
| 1           | 1             | N            |
| 2           | 1             | Y            |
| 2           | 2             | N            |
| 3           | 3             | N            |
| 4           | 2             | N            |
| 4           | 3             | Y            |
| 4           | 4             | N            |
+-------------+---------------+--------------+
输出：
+-------------+---------------+
| employee_id | department_id |
+-------------+---------------+
| 1           | 1             |
| 2           | 1             |
| 3           | 3             |
| 4           | 3             |
+-------------+---------------+
解释：
- 员工 1 的直属部门是 1
- 员工 2 的直属部门是 1
- 员工 3 的直属部门是 3
- 员工 4 的直属部门是 3
```





第一种情况：

```sql
SELECT e.employee_id, e.department_id 
FROM Employee e
WHERE e.primary_flag="Y";
```



第二种情况：

```sql
SELECT employee_id  FROM Employee 
GROUP BY employee_id HAVING COUNT(department_id)=1;
```



```sqlW
SELECT e.employee_id, e.department_id 
FROM Employee e
WHERE e.primary_flag="Y"
OR (e.employee_id in (SELECT employee_id  FROM Employee 
GROUP BY employee_id HAVING COUNT(department_id)=1));  
```



```sql
with a as (
select 
employee_id,
department_id,
primary_flag,
count(*) over(partition by employee_id) cnt 
from Employee
)


select 
employee_id,department_id
from a 
where cnt = 1 or primary_flag = 'Y'
```





#### [610. 判断三角形](https://leetcode.cn/problems/triangle-judgement/)

**示例 1:**

```
输入: 
Triangle 表:
+----+----+----+
| x  | y  | z  |
+----+----+----+
| 13 | 15 | 30 |
| 10 | 20 | 15 |
+----+----+----+
输出: 
+----+----+----+----------+
| x  | y  | z  | triangle |
+----+----+----+----------+
| 13 | 15 | 30 | No       |
| 10 | 20 | 15 | Yes      |
+----+----+----+----------+
```





方法一: case when

```sql
SELECT 
    x,
    y,
    z,
    CASE
        WHEN x + y > z AND x + z > y AND y + z > x THEN 'Yes'
        ELSE 'No'
    END AS 'triangle'
FROM
    triangle;
```



方法二：IF

```sql
SELECT x,y,z , IF(x+y>z AND x+z>y AND y+z>x,"Yes","No") AS triangle
FROM triangle;
```



### 窗口函数：

- `OVER` 是用于定义窗口规范（Window Specification）的子句，它通常与窗口函数一起使用。窗口规范定义了窗口函数操作的数据集合，即指定在哪个范围内执行窗口函数。`

- 语法结构如下：

  ```sql
  SELECT
    column1,
    column2,
    window_function() OVER (PARTITION BY partition_column ORDER BY order_column ROWS BETWEEN start_row AND end_row) AS result_column
  FROM
    your_table;
  ```

  - **`PARTITION BY partition_column`**: 按照指定的列进行分区，窗口函数将在每个分区内独立计算。如果省略 `PARTITION BY`，整个结果集将被视为一个分区。
  - **`ORDER BY order_column`**: 对分区内的行进行排序，以便窗口函数按照指定的顺序执行。如果不提供 `ORDER BY`，窗口函数将在分区内的未排序行上执行。
  - **`ROWS BETWEEN start_row AND end_row`**: 定义窗口的行范围，指定窗口函数计算的行的范围。常见的选项包括 `UNBOUNDED PRECEDING`（从分区的第一行开始），`CURRENT ROW`（当前行），以及 `n PRECEDING` 和 `n FOLLOWING`（相对于当前行的前n行和后n行）等。

  窗口规范中的 `OVER` 子句使得窗口函数能够在指定的数据子集上执行计算，而不是在整个结果集上进行计算。

  举个例子：

  ```sql
  SELECT
    department_id,
    salary,
    AVG(salary) OVER (PARTITION BY department_id ORDER BY salary) AS avg_salary
  FROM
    Employee;
  ```

  在这个例子中，`OVER (PARTITION BY department_id ORDER BY salary)` 定义了一个窗口规范，它将结果集按照 `department_id` 进行分区，并在每个分区内按照 `salary` 进行排序。然后，`AVG(salary)` 窗口函数将在每个分区内计算薪水的平均值。

- `LEAD` 和 `LAG` 是 SQL 中的窗口函数，它们用于访问结果集中当前行之前或之后的行的值

- **LEAD:**

  - `LEAD(column, offset, default)` 返回在当前行之后指定偏移量处的行的值。
  - `column`: 要获取值的列。
  - `offset`: 行偏移量，指定要查找的行数。默认为 1。
  - `default`: 如果未找到指定偏移量的行，则返回的默认值。

  例子：

  ```
  sqlCopy codeSELECT num, LEAD(num, 1) OVER (ORDER BY id) AS next_num
  FROM Logs;
  ```

- **LAG:**

  - `LAG(column, offset, default)` 返回在当前行之前指定偏移量处的行的值。
  - `column`: 要获取值的列。
  - `offset`: 行偏移量，指定要查找的行数。默认为 1。
  - `default`: 如果未找到指定偏移量的行，则返回的默认值。

  例子：

  ```
  sqlCopy codeSELECT num, LAG(num, 1) OVER (ORDER BY id) AS prev_num
  FROM Logs;
  ```



- 使用 `PARTITION BY` 子句时，你将在结果集中创建多个独立的窗口，每个窗口都是根据指定的列进行分区的。窗口函数将在每个分区内进行计算。

  ```
  sqlCopy codeSELECT
     department_id,
     employee_id,
     salary,
     AVG(salary) OVER (PARTITION BY department_id) as avg_salary
  FROM Employee;
  ```

  `AVG(salary) OVER (PARTITION BY department_id)` 计算了每个部门内员工薪水的平均值，因此窗口根据 `department_id` 进行分区。

- 没有使用 `PARTITION BY`，则整个结果集将被视为一个窗口。窗口函数将在整个结果集上执行计算。

  ```
  sqlCopy codeSELECT
     employee_id,
     department_id,
     salary,
     AVG(salary) OVER () as overall_avg_salary
  FROM Employee;
  ```

  在这个例子中，`AVG(salary) OVER ()` 计算了整个结果集中员工薪水的平均值，因为没有指定分区。

- 在 SQL 中，你不能在 `WHERE` 子句中直接引用使用窗口函数计算的别名。这是因为 `WHERE` 子句在查询执行的早期阶段进行筛选，而窗口函数的计算是在结果集形成的晚期阶段进行的。希望筛选出 `total_weight` 小于等于 1000 的行，你可以使用子查询或通用表表达式 (CTE) 来实现。以下是使用子查询的一个例子：

  

  

  

### [180. 连续出现的数字](https://leetcode.cn/problems/consecutive-numbers/)

找出所有至少连续出现三次的数字。

返回的结果表中的数据可以按 **任意顺序** 排列。

结果格式如下面的例子所示：

**示例 1:**

```sql
输入：
Logs 表：
+----+-----+
| id | num |
+----+-----+
| 1  | 1   |
| 2  | 1   |
| 3  | 1   |
| 4  | 2   |
| 5  | 1   |
| 6  | 2   |
| 7  | 2   |
+----+-----+
输出：
Result 表：
+-----------------+
| ConsecutiveNums |
+-----------------+
| 1               |
+-----------------+
解释：1 是唯一连续出现至少三次的数字。
```



难点：

- 连续出现的数字的检测
- distinct 和 多表连接的条件
- 窗口函数



```sql
SELECT DISTINCT
    l1.Num AS ConsecutiveNums
FROM
    Logs l1,
    Logs l2,
    Logs l3
WHERE
    l1.Id = l2.Id - 1
    AND l2.Id = l3.Id - 1
    AND l1.Num = l2.Num
    AND l2.Num = l3.Num
;

```



```sql
WITH ConsecutiveNums AS (
    SELECT
        num,
        LEAD(num, 1) OVER (ORDER BY id) AS next_num,
        LAG(num, 1) OVER (ORDER BY id) AS prev_num
    FROM Logs
)

SELECT DISTINCT num AS ConsecutiveNums
FROM ConsecutiveNums
WHERE num = next_num AND num = prev_num;
```





方法 ：
row_number() over([partition by value_expression,...n] order by columnName)

- `ROW_NUMBER()` 是 SQL 中的窗口函数之一，用于为查询结果集中的每一行分配一个唯一的行号。

```sql
SELECT id, num,
ROW_NUMBER() OVER (PARTITION BY num ORDER BY id) AS SerialGroup
```

![image-20231007103603923](D:\kylec\Documents\Programming Files\笔记\算法与数据结构体系学习班\media\image-20231007103603923.png)



- 两个列（**SerialNum**，**SerialGroup**）对应相减，只要连续，相减得到的值是一样的。不连续相减得到的值也不同。



```sql
SELECT id, num , row_number() over(order by id) - row_number() over (partition by num order by id) as SerialNumberSubGroup
FROM Lags
```



![image-20231007103855807](D:\kylec\Documents\Programming Files\笔记\算法与数据结构体系学习班\media\image-20231007103855807.png)

\

```sql
SELECT num, count(num) as SerialCount 
FROM (
	SELECT id, num , row_number() over(order by id) - row_number() over (partition by num order by id) as SerialNumber
FROM Lags
) as sub 
GROUP BY num, SerialNUmberSubGroup HAVING COUNT(num)>=3
```



```sql
SELECT DISTINCT Num FROM (
SELECT Num,COUNT(1) as SerialCount FROM 
(SELECT Id,Num,
row_number() over(order by id) -
ROW_NUMBER() over(partition by Num order by Id) as SerialNumberSubGroup
FROM ContinueNumber) as Sub
GROUP BY Num,SerialNumberSubGroup HAVING COUNT(1) >= 3) as Result
```





拓展结论：

- 如果一个num连续出现时，那么它出现的[真实序列]-它出现的次数一定是个定值。
  - 第一步重排的serialNum表示：这个num出现的 `真实序列`，也就是说在原始数据中，它是第几个出现的；
  - 第二步重排的serialGroup表示：这个num是第几次出现的；



### [1164. 指定日期的产品价格](https://leetcode.cn/problems/product-price-at-a-given-date/)

编写一个解决方案，找出在 `2019-08-16` 时全部产品的价格，假设所有产品在修改前的价格都是 `10` **。**

以 **任意顺序** 返回结果表。

结果格式如下例所示。

 

**示例 1:**

```sql
输入：
Products 表:
+------------+-----------+-------------+
| product_id | new_price | change_date |
+------------+-----------+-------------+
| 1          | 20        | 2019-08-14  |
| 2          | 50        | 2019-08-14  |
| 1          | 30        | 2019-08-15  |
| 1          | 35        | 2019-08-16  |
| 2          | 65        | 2019-08-17  |
| 3          | 20        | 2019-08-18  |
+------------+-----------+-------------+
输出：
+------------+-------+
| product_id | price |
+------------+-------+
| 2          | 50    |
| 1          | 35    |
| 3          | 10    |
+------------+-------+
```





#### 方法：`left join` 和 `ifnull`

1. 找出所有的产品：

```sql
select distinct product_id from products 
```

2. 找到 `2019-08-16` 前所有有改动的产品的最新价格。
   - 注意：不能直接select new_price, 因为没法把最大的日期和价格同时查出来，需要外面再套一层

```sql
SELECT product_id, max(change_date) AS change_date
FROM products
WHERE change_date<="2019-08-16"
GROUP BY product_id;
```

![image-20231007113909201](D:\kylec\Documents\Programming Files\笔记\算法与数据结构体系学习班\media\image-20231007113909201.png)

```sql
SELECT product_id,new_price 
FROM Products WHERE (product_id,change_date) 
IN (
SELECT product_id, max(change_date) AS change_date
FROM products
WHERE change_date<="2019-08-16"
GROUP BY product_id
);
```

![image-20231007114159340](D:\kylec\Documents\Programming Files\笔记\算法与数据结构体系学习班\media\image-20231007114159340.png)

3. 使用 `left join` 得到所有产品的最新价格，如果没有设置为 `10`



```sql
SELECT
    p1.product_id,
    IFNULL(p2.new_price, 10) AS price
FROM (
    SELECT DISTINCT product_id
    FROM products
) p1
LEFT JOIN (
    SELECT
        product_id,
        new_price
    FROM Products
    WHERE (product_id, change_date) IN (
        SELECT
            product_id,
            MAX(change_date) AS change_date
        FROM products
        WHERE change_date <= '2019-08-16'
        GROUP BY product_id
    )
) p2 ON p1.product_id = p2.product_id;
```



### [1204. 最后一个能进入巴士的人](https://leetcode.cn/problems/last-person-to-fit-in-the-bus/)

```
turn 决定了候车乘客上巴士的顺序，其中 turn=1 表示第一个上巴士，turn=n 表示最后一个上巴士。
weight 表示候车乘客的体重，以千克为单位。
```

有一队乘客在等着上巴士。然而，巴士有`1000` **千克** 的重量限制，所以其中一部分乘客可能无法上巴士。

编写解决方案找出 **最后一个** 上巴士且不超过重量限制的乘客，并报告 `person_name` 。题目测试用例确保顺位第一的人可以上巴士且不会超重。

返回结果格式如下所示。

 

**示例 1：**

```sql
输入：
Queue 表
+-----------+-------------+--------+------+
| person_id | person_name | weight | turn |
+-----------+-------------+--------+------+
| 5         | Alice       | 250    | 1    |
| 4         | Bob         | 175    | 5    |
| 3         | Alex        | 350    | 2    |
| 6         | John Cena   | 400    | 3    |
| 1         | Winston     | 500    | 6    |
| 2         | Marie       | 200    | 4    |
+-----------+-------------+--------+------+
输出：
+-------------+
| person_name |
+-------------+
| John Cena   |
+-------------+
解释：
为了简化，Queue 表按 turn 列由小到大排序。
+------+----+-----------+--------+--------------+
| Turn | ID | Name      | Weight | Total Weight |
+------+----+-----------+--------+--------------+
| 1    | 5  | Alice     | 250    | 250          |
| 2    | 3  | Alex      | 350    | 600          |
| 3    | 6  | John Cena | 400    | 1000         | (最后一个上巴士)
| 4    | 2  | Marie     | 200    | 1200         | (无法上巴士)
| 5    | 4  | Bob       | 175    | ___          |
| 6    | 1  | Winston   | 500    | ___          |
+------+----+-----------+--------+--------------+
```



使用窗口函数;

```sql
SELECT person_name
FROM (
    SELECT *,
           SUM(weight) OVER (ORDER BY turn) AS total_weight
    FROM Queue
) q
WHERE q.total_weight <= 1000
ORDER BY turn DESC
LIMIT 1;

```

避免使用子查询：



```sql
WITH RankedQueue AS (
    SELECT
        person_id,
        person_name,
        SUM(weight) OVER (ORDER BY turn) AS total_weight,
        turn,
        ROW_NUMBER() OVER (ORDER BY turn DESC) AS rnk
    FROM Queue
)
SELECT person_name
FROM RankedQueue
WHERE total_weight <= 1000 AND rnk = 1;
```





### [1907. 按分类统计薪水](https://leetcode.cn/problems/count-salary-categories/)

查询每个工资类别的银行账户数量。 工资类别如下：

- `"Low Salary"`：所有工资 **严格低于** `20000` 美元。
- `"Average Salary"`： **包含** 范围内的所有工资 `[$20000, $50000]` 。
- `"High Salary"`：所有工资 **严格大于** `50000` 美元。

结果表 **必须** 包含所有三个类别。 如果某个类别中没有帐户，则报告 `0` 。

按 **任意顺序** 返回结果表。

查询结果格式如下示例。

 

**示例 1：**

```sql
输入：
Accounts 表:
+------------+--------+
| account_id | income |
+------------+--------+
| 3          | 108939 |
| 2          | 12747  |
| 8          | 87709  |
| 6          | 91796  |
+------------+--------+
输出：
+----------------+----------------+
| category       | accounts_count |
+----------------+----------------+
| Low Salary     | 1              |
| Average Salary | 0              |
| High Salary    | 3              |
+----------------+----------------+
解释：
低薪: 有一个账户 2.
中等薪水: 没有.
高薪: 有三个账户，他们是 3, 6和 8.
```

难点：

- 建立category column ，原表格中没有这一列
- 处理0，为0的一组category 不会显示
- `UNION ALL` 是 SQL 中的一个操作符，用于合并两个或多个查询的结果集，并返回一个包含所有行的结果集。它的主要特点是允许包含重复的行，即使在不同的子查询中有相同的行，也会全部保留。相对应的，`UNION` 操作符则会删除重复的行，只保留一次。



生成一个包含所有category的表

```sql
WITH categories AS (
SELECT "Low Salary" AS category
UNION ALL  
    SELECT "Average Salary"
UNION ALL
    SELECT "High Salary"
)

```

对Accounts表的salary进行统计，聚合结果

```sql
SELECT 
    CASE 
      WHEN income<20000 THEN "Low Salary"
      WHEN income>=20000 AND income<50000 THEN "Average Salary"
      WHEN income>=50000 THEN "High Salary"
      END AS category,
     COUNT(*) AS accounts_count
FROM Accounts
GROUP BY category;
```

LEFT JOIN 连接两个表

```sql
SELECT c.category ,ifnull(a.accounts_count,0) AS accounts_count
FROM categories c
LEFT JOIN (
    SELECT 
    CASE 
      WHEN income<20000 THEN "Low Salary"
      WHEN income>=20000 AND income<=50000 THEN "Average Salary"
      WHEN income>50000 THEN "High Salary"
      END AS category,
     COUNT(*) AS accounts_count
FROM Accounts
GROUP BY category
) a
ON c.category=a.category
;
```



#### 方法二：

```sql
SELECT 
    'Low Salary' AS category,
    SUM(CASE WHEN income < 20000 THEN 1 ELSE 0 END) AS accounts_count
FROM 
    Accounts
    
UNION
SELECT  
    'Average Salary' category,
    SUM(CASE WHEN income >= 20000 AND income <= 50000 THEN 1 ELSE 0 END) 
    AS accounts_count
FROM 
    Accounts

UNION
SELECT 
    'High Salary' category,
    SUM(CASE WHEN income > 50000 THEN 1 ELSE 0 END) AS accounts_count
FROM 
    Accounts
```



## Day 7

### [1978. 上级经理已离职的公司员工](https://leetcode.cn/problems/employees-whose-manager-left-the-company/)



查找这些员工的id，他们的薪水严格少于`$30000` 并且他们的上级经理已离职。当一个经理离开公司时，他们的信息需要从员工表中删除掉，但是表中的员工的`manager_id` 这一列还是设置的离职经理的id 。

返回的结果按照`employee_id `从小到大排序。

查询结果如下所示：

 

**示例：**

```sql
输入：
Employees table:
+-------------+-----------+------------+--------+
| employee_id | name      | manager_id | salary |
+-------------+-----------+------------+--------+
| 3           | Mila      | 9          | 60301  |
| 12          | Antonella | null       | 31000  |
| 13          | Emery     | null       | 67084  |
| 1           | Kalel     | 11         | 21241  |
| 9           | Mikaela   | null       | 50937  |
| 11          | Joziah    | 6          | 28485  |
+-------------+-----------+------------+--------+
输出：
+-------------+
| employee_id |
+-------------+
| 11          |
+-------------+

解释：
薪水少于 30000 美元的员工有 1 号(Kalel) 和 11号 (Joziah)。
Kalel 的上级经理是 11 号员工，他还在公司上班(他是 Joziah )。
Joziah 的上级经理是 6 号员工，他已经离职，因为员工表里面已经没有 6 号员工的信息了，它被删除了。
```



考察子查询：

```sql
SELECT 
	employee_id 
FROM 
	Employees 
WHERE 
	salary<30000
AND 
	manager_id NOT IN (
        SELECT employee_id FROM Employees);
```



### [626. 换座位](https://leetcode.cn/problems/exchange-seats/)

编写解决方案来交换每两个连续的学生的座位号。如果学生的数量是奇数，则最后一个学生的id不交换。

按 `id` **升序** 返回结果表。

查询结果格式如下所示。

 

**示例 1:**

```sql
输入: 
Seat 表:
+----+---------+
| id | student |
+----+---------+
| 1  | Abbot   |
| 2  | Doris   |
| 3  | Emerson |
| 4  | Green   |
| 5  | Jeames  |
+----+---------+
输出: 
+----+---------+
| id | student |
+----+---------+
| 1  | Doris   |
| 2  | Abbot   |
| 3  | Green   |
| 4  | Emerson |
| 5  | Jeames  |
+----+---------+
解释:
请注意，如果学生人数为奇数，则不需要更换最后一名学生的座位。
```





难点：

- 连续的数字
- 交换
- 奇数 
  - id%2



#### **算法**

对于所有座位 id 是奇数的学生，修改其 id 为 id+1，如果最后一个座位 id 也是奇数，则最后一个座位 id 不修改。对于所有座位 id 是偶数的学生，修改其 id 为 id-1。



```sql
SELECT id, IF(id%2=1,LEAD(student,1,student) OVER (ORDER BY id), LAG(student,1) OVER (ORDER BY id)) student
FROM Seat;

```



#### 解法2：

```sql
select 
    if(id%2=0,
        id-1,
        if(id=(select count(distinct id) from seat),
            id,
            id+1)) 
    as id,student 
from seat 
order by id;

```



### [1341. 电影评分](https://leetcode.cn/problems/movie-rating/)

请你编写一个解决方案：

- 查找评论电影数量最多的用户名。如果出现平局，返回字典序较小的用户名。
- 查找在 `February 2020` **平均评分最高** 的电影名称。如果出现平局，返回字典序较小的电影名称。

**字典序** ，即按字母在字典中出现顺序对字符串排序，字典序较小则意味着排序靠前。

返回结果格式如下例所示。

 

**示例 1：**

```sql
输入：
Movies 表：
+-------------+--------------+
| movie_id    |  title       |
+-------------+--------------+
| 1           | Avengers     |
| 2           | Frozen 2     |
| 3           | Joker        |
+-------------+--------------+
Users 表：
+-------------+--------------+
| user_id     |  name        |
+-------------+--------------+
| 1           | Daniel       |
| 2           | Monica       |
| 3           | Maria        |
| 4           | James        |
+-------------+--------------+
MovieRating 表：
+-------------+--------------+--------------+-------------+
| movie_id    | user_id      | rating       | created_at  |
+-------------+--------------+--------------+-------------+
| 1           | 1            | 3            | 2020-01-12  |
| 1           | 2            | 4            | 2020-02-11  |
| 1           | 3            | 2            | 2020-02-12  |
| 1           | 4            | 1            | 2020-01-01  |
| 2           | 1            | 5            | 2020-02-17  | 
| 2           | 2            | 2            | 2020-02-01  | 
| 2           | 3            | 2            | 2020-03-01  |
| 3           | 1            | 3            | 2020-02-22  | 
| 3           | 2            | 4            | 2020-02-25  | 
+-------------+--------------+--------------+-------------+
输出：
Result 表：
+--------------+
| results      |
+--------------+
| Daniel       |
| Frozen 2     |
+--------------+
解释：
Daniel 和 Monica 都点评了 3 部电影（"Avengers", "Frozen 2" 和 "Joker"） 但是 Daniel 字典序比较小。
Frozen 2 和 Joker 在 2 月的评分都是 3.5，但是 Frozen 2 的字典序比较小。
```





#### 知识点：

- 两个查询结果：UNION ALL

- COUNT(*) 和AVG(rating) 的结果直接放在ORDER  BY 后面，而不是select

- ```sql
  created_at LIKE '2020-02%'
  ```



```sql
-- First case: Movie ratings and user information
(SELECT b.name AS results
 FROM MovieRating a
 JOIN Users b ON a.user_id = b.user_id
 GROUP BY a.user_id
 ORDER BY COUNT(a.user_id) DESC, b.name ASC
 LIMIT 1)
UNION ALL

-- Second case: Movie information for a specific month and average rating
(SELECT b.title AS results
 FROM MovieRating a
 JOIN Movies b ON a.movie_id = b.movie_id
 WHERE date_format(a.created_at,"%Y-%m")="2020-02"
 GROUP BY a.movie_id
 ORDER BY AVG(a.rating) DESC, b.title ASC
 LIMIT 1);

```

窗口函数：RANK ()

- `RANK()` 函数是 SQL 中的一种窗口函数，用于为结果集中的每个唯一行分配一个唯一的排名。它通常用于根据指定的顺序对行进行排名。其一般语法如下：

  ```sql
  sqlCopy code
  RANK() OVER (PARTITION BY partition_expression ORDER BY sort_expression [ASC | DESC], ...)
  ```

  - `PARTITION BY`：将结果集分为窗口，`RANK()` 函数将应用于这些窗口。如果省略 `PARTITION BY` 子句，该函数将整个结果集视为一个窗口。
  - `ORDER BY`：指定行在每个窗口内的排名顺序。可以在 `ORDER BY` 子句中有多个列，以定义排序标准。可选的 `ASC`（升序）或 `DESC`（降序）关键字确定排序顺序

  

```sql
SELECT results
FROM (
  SELECT name AS results, RANK() OVER(ORDER BY COUNT(*) DESC, name) AS RANKING
  FROM Users
  INNER JOIN MovieRating USING(user_id)
  GROUP BY user_id
  UNION ALL
  SELECT title AS results, RANK() OVER(ORDER BY AVG(rating) DESC, title) AS RANKING
  FROM MovieRating
  INNER JOIN Movies USING(movie_id)
  WHERE DATE_FORMAT(created_at, '%Y-%m') = '2020-02'
  GROUP BY movie_id
) T
WHERE T.RANKING = 1
```





### [1321. 餐馆营业额变化增长](https://leetcode.cn/problems/restaurant-growth/)

你是餐馆的老板，现在你想分析一下可能的营业额变化增长（每天至少有一位顾客）。

计算以 7 天（某日期 + 该日期前的 6 天）为一个时间段的顾客消费平均值。`average_amount` 要 **保留两位小数。**

结果按 `visited_on` **升序排序**。

返回结果格式的例子如下。

 

**示例 1:**

```sql
输入：
Customer 表:
+-------------+--------------+--------------+-------------+
| customer_id | name         | visited_on   | amount      |
+-------------+--------------+--------------+-------------+
| 1           | Jhon         | 2019-01-01   | 100         |
| 2           | Daniel       | 2019-01-02   | 110         |
| 3           | Jade         | 2019-01-03   | 120         |
| 4           | Khaled       | 2019-01-04   | 130         |
| 5           | Winston      | 2019-01-05   | 110         | 
| 6           | Elvis        | 2019-01-06   | 140         | 
| 7           | Anna         | 2019-01-07   | 150         |
| 8           | Maria        | 2019-01-08   | 80          |
| 9           | Jaze         | 2019-01-09   | 110         | 
| 1           | Jhon         | 2019-01-10   | 130         | 
| 3           | Jade         | 2019-01-10   | 150         | 
+-------------+--------------+--------------+-------------+
输出：
+--------------+--------------+----------------+
| visited_on   | amount       | average_amount |
+--------------+--------------+----------------+
| 2019-01-07   | 860          | 122.86         |
| 2019-01-08   | 840          | 120            |
| 2019-01-09   | 840          | 120            |
| 2019-01-10   | 1000         | 142.86         |
+--------------+--------------+----------------+
解释：
第一个七天消费平均值从 2019-01-01 到 2019-01-07 是restaurant-growth/restaurant-growth/ (100 + 110 + 120 + 130 + 110 + 140 + 150)/7 = 122.86
第二个七天消费平均值从 2019-01-02 到 2019-01-08 是 (110 + 120 + 130 + 110 + 140 + 150 + 80)/7 = 120
第三个七天消费平均值从 2019-01-03 到 2019-01-09 是 (120 + 130 + 110 + 140 + 150 + 80 + 110)/7 = 120
第四个七天消费平均值从 2019-01-04 到 2019-01-10 是 (130 + 110 + 140 + 150 + 80 + 110 + 130 + 150)/7 = 142.86
```



#### 知识点：



在 SQL 窗口函数中，`ROWS BETWEEN` 子句用于定义窗口框架

1. **PRECEDING 和 FOLLOWING：** PRECEDING` 表示之前的行数，FOLLOWING` 表示之后的行数。例如，`ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING` 表示在当前行之前和之后各包含一行。

2. **UNBOUNDED：** 你可以使用 `UNBOUNDED` 表示没有限制。例如，`ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` 表示从结果集的开头到当前行。

3. **CURRENT ROW：** 这表示只包括当前行。

   



在 SQL 窗口函数中，`RANGE BETWEEN INTERVAL` 的用法用于指定窗口框架，表示在时间上的范围

- `RANGE`: 这表示你正在定义一个范围作为窗口框架，而不是特定的行数。
- `BETWEEN`: 这是一个用于指定范围边界的关键字。
- `INTERVAL 6 DAY PRECEDING`: 这表示在当前行之前的 6 天。这部分定义了窗口的起始位置。
- `AND CURRENT ROW`: 这表示窗口的结束位置是当前行。这部分定义了窗口的结束位置。



具体有如下这些：

当前行 - current row
之前的行 - preceding
之后的行 - following
无界限 - unbounded
表示从前面的起点 - unbounded preceding
表示到后面的终点 - unbounded following
举例理解一下：

取当前行和前五行：ROWS between 5 preceding and current row --共6行
取当前行和后五行：ROWS between current row and 5 following --共6行
取前五行和后五行：ROWS between 5 preceding and 5 folowing --共11行



陷阱：

- 不要使用AVG，如果只有5条数据，那么窗口函数会/5，不管那天有没有记录，求平均的时候都要算上，而不是只算有记录的7天
- 存在着同一用户在某日多次消费的情况，窗口照旧向前取6天就无法覆盖被挤出去的数据了



陷阱2：

```sql
SELECT a.visited_on,

  SUM(amount) OVER (ORDER BY visited_on ASC RANGE INTERVAL 6 DAY PRECEDING) AS total_amount
  
FROM
 (select visited_on,sum(amount) as amount from Customer GROUP BY visited_on) a
```



陷阱1：

```sql
SELECT visited_on, total_amount as amount, ROUND(total_amount/7,2) AS average_amount

FROM (

SELECT a.visited_on,

  SUM(amount) OVER (ORDER BY visited_on ASC RANGE INTERVAL 6 DAY PRECEDING) AS total_amount
  
FROM
 (select visited_on,sum(amount) as amount from Customer GROUP BY visited_on) a) b 


WHERE DATEDIFF(visited_on, (SELECT MIN(visited_on) FROM Customer)) >= 6;
```





### [602. 好友申请 II ：谁有最多的好友](https://leetcode.cn/problems/friend-requests-ii-who-has-the-most-friends/)

编写解决方案，找出拥有最多的好友的人和他拥有的好友数目。

生成的测试用例保证拥有最多好友数目的只有 1 个人。

查询结果格式如下例所示。

 

**示例 1：**

```
输入：
RequestAccepted 表：
+--------------+-------------+-------------+
| requester_id | accepter_id | accept_date |
+--------------+-------------+-------------+
| 1            | 2           | 2016/06/03  |
| 1            | 3           | 2016/06/08  |
| 2            | 3           | 2016/06/08  |
| 3            | 4           | 2016/06/09  |
+--------------+-------------+-------------+
输出：
+----+-----+
| id | num |
+----+-----+
| 3  | 3   |
+----+-----+
解释：
编号为 3 的人是编号为 1 ，2 和 4 的人的好友，所以他总共有 3 个好友，比其他人都多。
```

 

**进阶：**在真实世界里，可能会有多个人拥有好友数相同且最多，你能找到所有这些人吗？



#### 算法

成为朋友是一个双向的过程，所以如果一个人接受了另一个人的请求，他们两个都会多拥有一个朋友。

所以我们可以将 requester_id 和 accepter_id 联合起来，然后统计每个人出现的次数。



```sql
SELECT a.ids as id, count(*) as num FROM

( select requester_id as ids from RequestAccepted 
        union all
  select accepter_id as ids from RequestAccepted) a 
  GROUP BY id order by num desc limit 1;
```



## Day 8



### [585. 2016年的投资](https://leetcode.cn/problems/investments-in-2016/)



编写解决方案报告 2016 年 (`tiv_2016`) 所有满足下述条件的投保人的投保金额之和：

- 他在 2015 年的投保额 (`tiv_2015`) 至少跟一个其他投保人在 2015 年的投保额相同。
- 他所在的城市必须与其他投保人都不同（也就是说 (`lat, lon`) 不能跟其他任何一个投保人完全相同）。

`tiv_2016` 四舍五入的 **两位小数** 。

查询结果格式如下例所示。

 

**示例 1：**

```sql
输入：
Insurance 表：
+-----+----------+----------+-----+-----+
| pid | tiv_2015 | tiv_2016 | lat | lon |
+-----+----------+----------+-----+-----+
| 1   | 10       | 5        | 10  | 10  |
| 2   | 20       | 20       | 20  | 20  |
| 3   | 10       | 30       | 20  | 20  |
| 4   | 10       | 40       | 40  | 40  |
+-----+----------+----------+-----+-----+
输出：
+----------+
| tiv_2016 |
+----------+
| 45.00    |
+----------+
解释：
表中的第一条记录和最后一条记录都满足两个条件。
tiv_2015 值为 10 与第三条和第四条记录相同，且其位置是唯一的。

第二条记录不符合任何一个条件。其 tiv_2015 与其他投保人不同，并且位置与第三条记录相同，这也导致了第三条记录不符合题目要求。
因此，结果是第一条记录和最后一条记录的 tiv_2016 之和，即 45 。
```



#### 窗口函数解法：                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     



```sql
SELECT ROUND(SUM(tiv_2016),2) AS tiv_2016 
FROM ( 
    SELECT 
    tiv_2016,
    COUNT(*) OVER (PARTITION BY lat,lon ) AS count_lat_lon,
    COUNT(*) OVER (PARTITION BY tiv_2015) AS count_tiv_2015
    FROM Insurance 
	) a 
where a.count_lat_lon=1 AND a.count_tiv_2015>1;
```



```sql
SELECT ROUND(SUM(a.tiv_2016), 2) AS tiv_2016
FROM Insurance a
WHERE 
    (a.lat, a.lon) NOT IN (
        SELECT b.lat, b.lon
        FROM Insurance b
        WHERE a.pid <> b.pid
    )
    AND a.tiv_2015 IN (
        SELECT c.tiv_2015
        FROM Insurance c
        WHERE a.pid <> c.pid
    );

```





### [185. 部门工资前三高的所有员工](https://leetcode.cn/problems/department-top-three-salaries/)



公司的主管们感兴趣的是公司每个部门中谁赚的钱最多。一个部门的 **高收入者** 是指一个员工的工资在该部门的 **不同** 工资中 **排名前三** 。

编写解决方案，找出每个部门中 **收入高的员工** 。

以 **任意顺序** 返回结果表。

返回结果格式如下所示。

 

**示例 1:**

```sql
输入: 
Employee 表:
+----+-------+--------+--------------+
| id | name  | salary | departmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 85000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
| 5  | Janet | 69000  | 1            |
| 6  | Randy | 85000  | 1            |
| 7  | Will  | 70000  | 1            |
+----+-------+--------+--------------+
Department  表:
+----+-------+
| id | name  |
+----+-------+
| 1  | IT    |
| 2  | Sales |
+----+-------+
输出: 
+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| IT         | Joe      | 85000  |
| IT         | Randy    | 85000  |
| IT         | Will     | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |
+------------+----------+--------+
解释:
在IT部门:
- Max的工资最高
- 兰迪和乔都赚取第二高的独特的薪水
- 威尔的薪水是第三高的

在销售部:
- 亨利的工资最高
- 山姆的薪水第二高
- 没有第三高的工资，因为只有两名员工
```



#### 使用窗口函数：dense_rank()

`

**`RANK()` 函数：**

- `RANK()` 函数分配排名，并在出现相同数值时跳过排名。如果有两行具有相同的值，它们将被分配相同的排名，并且下一行将跳过相应数量的排名。
- 例如，如果两行具有相同的值并且都排在第一位，则下一行将被分配排名为第三位。

**`DENSE_RANK()` 函数：**

- `DENSE_RANK()` 函数也分配排名，但在出现相同数值时不跳过排名。即使有两行具有相同的值，它们也将被分配相同的排名，而下一行将继续顺序分配排名。
- 例如，如果两行具有相同的值并且都排在第一位，则下一行将被分配排名为第二位。



```sql
SELECT a.name AS Department, b.name AS Employee, b.salary FROM Department a 
join (

SELECT *, DENSE_RANK() OVER (partition by departmentId ORDER BY salary DESC) as salary_rank
FROM Employee ) b on a.id=b.departmentId WHERE salary_rank<=3 ;
```



### [1667. 修复表中的名字](https://leetcode.cn/problems/fix-names-in-a-table/)



编写解决方案，修复名字，使得只有第一个字符是大写的，其余都是小写的。

返回按 `user_id` 排序的结果表。

返回结果格式示例如下。

 

**示例 1：**

```sql
输入：
Users table:
+---------+-------+
| user_id | name  |
+---------+-------+
| 1       | aLice |
| 2       | bOB   |
+---------+-------+
输出：
+---------+-------+
| user_id | name  |
+---------+-------+
| 1       | Alice |
| 2       | Bob   |
+---------+-------+
```



#### 考察字符串函数的用法：

- `CONCAT(str1, str2) :` 拼接字符串
- `UPPER(str) :` 字符串变成大写
- `LOWER(str) :` 字符串变成小写
- `LENGTH(str) :` 获取字符串的长度
- `LEFT(str,len) :` 获取字符串左边 len 个字符
- `RIGHT(str,len) :` 获取字符串右边 len 个字符
- `SUBSTR(str,start,len) :` 获取 str 中从 start 开始的 len 个字符



```sql
SELECT user_id, CONCAT(UPPER(LEFT(name,1)),LOWER(RIGHT(name,LENGTH(name)-1))) AS name
FROM Users
ORDER BY user_id;
```



```sql
select user_id,
CONCAT(Upper(left(name,1)),Lower(substring(name,2))) name 
from users 
order by user_id
```



#### 正则表达：

```sql
SELECT * FROM PATIENTS
WHERE CONDITIONS REGEXP '^DIAB1|\\sDIAB1'
```



```sql
SELECT * FROM 
patients WHERE conditions like "DIAB1%" or conditions LIKE "% DIAB1%";
```



- 这里使用了双反斜杠 `\\` 是因为在 SQL 字符串中需要转义反斜杠



