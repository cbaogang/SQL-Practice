

# 高频 SQL 50 题（基础版）

### 一. [584. 寻找用户推荐人](https://leetcode.cn/problems/find-customer-referee/)

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
SELECT name FROM Customer C WHERE C.referee_id <> 2 OR C.referee_id IS NULL;

```

解法 2：

```sql
SELECT c.name FROM Customer c WHERE c.id NOT IN (SELECT c1.id FROM Customer c1 WHERE c1.referee_id=2);

```

- `NOT IN` 是一个条件谓词，用于检查主查询的结果集中的值是否不在子查询的结果集中。
- 当使用 `NOT IN` 时，主查询的结果集中的值在子查询的结果集中找到匹配时，该行会被排除。
- `NOT IN` 的子查询通常返回一个包含单列值的结果集。

解法 3：

```sql
SELECT c.name FROM Customer c WHERE NOT EXISTS (SELECT c1.id FROM Customer c1 WHERE c1.referee_id=2 and c.id=c1.id);

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

   