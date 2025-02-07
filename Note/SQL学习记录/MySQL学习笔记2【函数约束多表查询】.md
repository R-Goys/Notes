## MySQL学习笔记

----

### 函数

#### 字符串函数

| 函数                             | 功能                                                        |
| :------------------------------- | ----------------------------------------------------------- |
| CONCAT(s1, s2, …, sn)            | 字符串拼接，将s1, s2, …, sn拼接成一个字符串                 |
| LOWER(str)                       | 将字符串全部转为小写                                        |
| UPPER(str)                       | 将字符串全部转为大写                                        |
| LPAD(str, n, pad)                | 左填充，用字符串pad对str的左边进行填充，达到n个字符串长度   |
| RPAD(str, n, pad)                | 右填充，用字符串pad对str的右边进行填充，达到n个字符串长度   |
| TRIM(str)                        | 去掉字符串头部和尾部的空格                                  |
| SUBSTRING(str, start, len)       | 截取字符串，从start开始截取len长度的字符串，**索引从1开始** |
| REPLACE(column, source, replace) | 替换字符串                                                  |

#### 数值函数

| 函数        | 功能                       |
| ----------- | -------------------------- |
| CEIL(x)     | 向上取整                   |
| FLOOR(x)    | 向下取整                   |
| MOD(x, y)   | 返回x/y的模                |
| RAND()      | 返回0~1内的随机数          |
| ROUND(x, y) | 对x的四舍五入，保留y位小数 |

#### 日期函数

| 函数                               | 功能                                              |
| ---------------------------------- | ------------------------------------------------- |
| CURDATE()                          | 返回当前日期                                      |
| CURTIME()                          | 返回当前时间                                      |
| NOW()                              | 返回当前日期和时间                                |
| YEAR(date)                         | 获取指定date的年份                                |
| MONTH(date)                        | 获取指定date的月份                                |
| DAY(date)                          | 获取指定date的日期                                |
| DATE_ADD(date, INTERVAL expr type) | 返回一个日期/时间值加上一个时间间隔expr后的时间值 |
| DATEDIFF(date1, date2)             | 返回起始时间date2和结束时间date1之间的天数        |

一个值得说的案例，查询员工入职天数，降序排序：

```sql
SELECT 
username, DATEDIFF(NOW(), entrydate) AS entrydays 
FROM users 
ORDER BY entrydays DESC;
```

#### 流程控制函数

1. **IF(value, t, f)**
   - **功能**：评估 `value` 的真假。
   - **用法**：如果 `value` 为真，返回 `t`；否则返回 `f`。
   - **示例**：
     ```sql
     SELECT IF(age >= 18, 'Adult', 'Minor') AS age_group FROM users;
     ```

2. **IFNULL(value1, value2)**
   - **功能**：检查 `value1` 是否为空。
   - **用法**：如果 `value1` 不为空，返回 `value1`；如果为空，返回 `value2`。
   - **示例**：
     ```sql
     SELECT IFNULL(nickname, username) AS display_name FROM users;
     ```

3. **CASE WHEN [ val1 ] THEN [ res1 ] … ELSE [ default ] END**
   - **功能**：执行多个条件的检查。
   - **用法**：如果 `val1` 为真，返回 `res1`，依此类推；如果都不满足，则返回 `default`。
   - **示例**：
     ```sql
     SELECT 
       CASE 
         WHEN score >= 90 THEN 'A'
         WHEN score >= 80 THEN 'B'
         ELSE 'C'
       END AS grade
     FROM scores;
     ```

4. **CASE [ expr ] WHEN [ val1 ] THEN [ res1 ] … ELSE [ default ] END**
   - **功能**：与第3个函数类似，但通过评估一个表达式来进行条件判断。
   - **用法**：如果 `expr` 等于 `val1`，返回 `res1`；如果不匹配，返回 `default`。
   - **示例**：
     ```sql
     SELECT 
       CASE user_type
         WHEN 'admin' THEN 'Administrator'
         WHEN 'user' THEN 'Regular User'
         ELSE 'Unknown'
       END AS user_role
     FROM users;
     ```

### 约束

| 约束条件                     | 关键字          | 说明                                                   |
|----------------------------|----------------|-------------------------------------------------------|
| 主键                         | PRIMARY KEY     | 唯一标识表中的每一行，主键列的值不能重复且不能为空。                |
| 自动增长                     | AUTO_INCREMENT  | 使字段在插入新行时自动增加（常用于主键）。                      |
| 不为空                       | NOT NULL        | 指定字段不能接受空值。                                  |
| 唯一                         | UNIQUE          | 确保字段的所有值都是唯一的，允许空值，但不能重复。              |
| 逻辑条件（条件一定要加上括号） | CHECK(约束条件) | 用于限制列中的值满足某些条件，必须使用括号以包围条件表达式。     |
| 默认值                       | DEFAULT         | 指定列的默认值，插入时如果不提供该列的值，则使用默认值。         |

- 值得一提的是，如果想要同时插入多个数据，只要有一条数据不满足约束，这条语句全部执行失败，就算其他数据符合约束条件，也不会插入进数据。
- 插入数据违反UNIQUE约束时，自增主键将会+1。

#### 删除/更新行为

ON DELETE/UPDATE

----

| 行为        | 说明                                                                                                                                              |
|:----------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| NO ACTION | 当在父表中删除或更新对应记录时，首先检查该记录是否有对应的外键。如果有，系统将不允许删除或更新（这与 `RESTRICT` 行为一致）。                        |
| RESTRICT  | 与 `NO ACTION` 一样，删除或更新前会检查是否存在对应外键，如果存在，则不允许进行操作。                                                                  |
| CASCADE | 当在父表中删除或更新对应记录时，如果该记录有对应的外键，系统会自动删除或更新子表中相关的记录。                                                    |
| SET NULL | 当在父表中删除或更新对应记录时，如果该记录有对应的外键，系统会将子表中该外键的值设置为 NULL（前提是该外键允许为 NULL）。                          |
| SET DEFAULT | 当父表的记录发生变更时，子表中的外键将被设为一个默认值（InnoDB 不支持此选项）。                                                                |

### 多表查询

| 关系类型          | 定义                                         | 示例描述                               | 数据库表示示例                                |
|------------------|--------------------------------------------|--------------------------------------|---------------------------------------------|
| 一对多 | 一个表中的一条记录可以关联到另一个表中的多条记录，而另一表中的每条记录只能关联到一条记录。 | 员工对应一个部门，部门有多个员工 | `department` 表与 `emp` 表，通过 `dept_id` 关联。 |
| 多对多 | 一个表中的多条记录可以关联到另一个表中的多条记录。通常需要一个关联表来实现这种关系。 | 一个学生可以选修多门课程，一门课程可以被多个学生选修。 | `students` 表与 `courses` 表，通过 `student_courses` 关联表。 |
| 一对一  | 一个表中的一条记录只能关联到另一个表中的一条记录，反之亦然。 | 每个用户有一条用户详情记录。           | `users` 表与 `user_details` 表，通过 `user_id` 关联。         |

- 一对一可以用来对单表进行拆分，结构更加清晰。

#### 1. 合并查询（笛卡尔积）

- **定义**：合并查询会显示两个表的所有组合结果，即笛卡尔积,  可以理解为1~9和0~9可以组合成多少个2位数

- **示例**：
  
  ```sql
  SELECT * FROM employee, dept;
  ```

#### 2. 内连接查询

- **定义**：内连接只返回两张表交集的部分，即满足连接条件的记录。
  
- **隐式内连接**：
  - 通过逗号分隔表名和用 `WHERE` 子句指定连接条件：
  ```sql
  SELECT e.name, d.name FROM employee AS e, dept AS d WHERE e.dept = d.id;
  ```

- **显式内连接**：
  - 使用 `JOIN` 关键字来显示连接关系，语法更清晰：
  ```sql
  SELECT e.name, d.name FROM employee AS e INNER JOIN dept AS d ON e.dept = d.id;
  ```

#### 3. 外连接查询

- **左外连接（LEFT OUTER JOIN）**：
  - 返回左表（employee）的所有数据，包含与右表（dept）交集部分的数据。
  ```sql
  SELECT e.*, d.name FROM employee AS e LEFT OUTER JOIN dept AS d ON e.dept = d.id;
  ```

- **右外连接（RIGHT OUTER JOIN）**：
  - 返回右表（dept）的所有数据，包含与左表（employee）交集部分的数据。
  ```sql
  SELECT d.name, e.* FROM employee AS e RIGHT OUTER JOIN dept AS d ON e.dept = d.id;
  ```

- **tips**:
  
  - 左连接可以查询到没有部门的员工，右连接可以查询到没有员工的部门。

#### 4. 自连接查询

- **定义**：当前表与自身进行连接查询，需要使用表别名。
  
- **语法**：
  ```sql
  SELECT 字段列表 FROM 表A 别名A JOIN 表A 别名B ON 条件 ...;
  ```

- **示例**：
  - 查询员工及其所属领导的名字：
  ```sql
  SELECT a.name, b.name FROM employee AS a, employee AS b WHERE a.manager = b.id;
  ```

  - 查询没有领导的员工：
  ```sql
  SELECT a.name, b.name FROM employee AS a LEFT JOIN employee AS b ON a.manager = b.id;
  ```

#### 5. 联合查询
```sql
SELECT 字段列表 FROM 表A
UNION [ALL]
SELECT 字段列表 FROM 表B;
```

##### 注意事项
1. **去重行为**：
   - `UNION`：会去掉重复的记录，只返回唯一的结果集。
   - `UNION ALL`：包括所有结果，保留重复记录。

2. **查询效率**：
   - 在某些情况下，使用`UNION`可能会比`OR`效率高。这是因为对于`UNION`，数据库可以对每个查询的结果集进行合并，而避免了对整个数据表的扫描。
   - 使用`OR`条件可能会导致某些情况下索引(现在还不知道这个是什么, 姑且先记一下)失效，从而导致查询性能下降。

3. **字段要求**：
   - 使用`UNION`或`UNION ALL`时，所有的SELECT查询返回的字段数和数据类型必须一致。

4. **执行顺序**：
   - 如果查询的顺序很重要，可以使用`ORDER BY`对最终的结果集进行排序，但需要注意`ORDER BY`只能在最后一个查询之后使用。

##### 使用示例
```sql
-- 查询两个表中用户的名字，去重
SELECT name FROM Customers
UNION
SELECT name FROM Employees;

-- 查询两个表中用户的名字，包括重复
SELECT name FROM Customers
UNION ALL
SELECT name FROM Employees;
```

#### 6.子查询

嵌套查询，也称为子查询，是一个查询嵌入在另一个查询内部。这种结构可以让我们在一个查询中利用另一个查询的结果。

----

##### 子查询的分类
1. **根据结果形式**：
   - **标量子查询**：返回单个值（如一个数字、一个字符串等）。
   - **列子查询**：返回一列，可以是多行。
   - **行子查询**：返回一行，包含多个列。
   - **表子查询**：返回多行多列的结果。
   
2. **根据位置**：
   - **WHERE之后**：用于过滤主查询的结果。
   - **FROM之后**：作为数据源提供给主查询。
   - **SELECT之后**：用于计算和生成字段值。

##### 常用的操作符
- **标量子查询**：常用操作符包括`=`, `<`, `>`, `>=`, `<=`。
- **列子查询**：常用操作符包括`IN`, `NOT IN`, `ANY`, `SOME`, `ALL`。
- **行子查询**：可以使用`=`, `<`, `>`, `IN`, `NOT IN`。
- **表子查询**：多用于`IN`操作符。

##### 示例分析
1. **标量子查询示例**：
   ```sql
   SELECT * 
   FROM employee 
   WHERE entrydate > ( #子查询返回一个常量
       SELECT entrydate 
       FROM employee 
       WHERE name = 'xxx'
   );
   ```
   
2. **列子查询示例**：
   
   ```sql
   SELECT * 
   FROM employee 
   WHERE dept 
   IN (	#子查询返回很多列,如果dept存在于查询的结果之中,则算满足条件.
       SELECT id 
       FROM dept 
       WHERE name = '销售部' OR name = '市场部'
   );
   ```
   
3. **行子查询示例**：
   
   ```sql
   SELECT * 
   FROM employee
   WHERE (salary, manager) = (   #查询一列,返回N行,如果这几行全部相等,那么则算满足条件,
       SELECT salary, manager 
       FROM employee 
       WHERE name = 'xxx'
   );
   ```
   
4. **表子查询示例**：
   
   ```sql
   SELECT e.*, d.* 
   FROM (  #查询一张表,从这张表中再进行条件过滤或者左连接
       SELECT * 
       FROM employee 
       WHERE entrydate > '2006-01-01'
   ) AS e
   LEFT JOIN dept AS d 
   ON e.dept = d.id;
   ```

​	`TIPS:表字查询也可以放在WHERE '字段' IN后面, 和列子查询类似,只要满足其中一列全部相等,那么则算满足条件`

#### 值得一提的案例

  ```sql
  SELECT 
      d.id, 	#此处可以理解为先从FROM中查询到了id，再将id传入子查询中，通过count得出人数
      d.name, 
      (
          SELECT 
          	COUNT(*) 
          FROM 
          	emp e 
          WHERE 
          	e.dept_id = d.id
      ) AS '人数'
  FROM 
  	dept d;
  ```

  



