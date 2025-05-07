## MySQL学习笔记5【SQL优化/视图/存储过程/触发器】

----

### SQL优化

------

#### 1. 插入数据优化

**普通插入：**
- **采用批量插入**：
  - 每次插入不建议超过1000条记录，这样可以减少事务开销，提高性能。
  - 示例：
  ```sql
  INSERT INTO tb_user (name, age) VALUES ('Alice', 25), ('Bob', 30), ...;
  ```

- **手动提交事务**：
  - 在插入过程中手动控制事务的开始与提交，以减少自动提交的次数。
  - 示例：
  ```sql
  START TRANSACTION;
  INSERT INTO tb_user (name, age) VALUES ('Alice', 25);
  ... 
  COMMIT;
  ```

- **顺序插入主键**：
  
  - 使用自增主键，避免随机插入造成的页分裂，提升插入速度。

**大批量插入：**
- **使用 `LOAD DATA INFILE`**：
  - 当需要插入大量数据时，通过文件导入的方式提升性能。
  - 客户端连接时加上参数 `--local-infile`：
  ```bash
  mysql --local-infile -u root -p
  ```

  - 设置全局参数：
  ```sql
  SET GLOBAL local_infile = 1;
  ```

  - 使用示例：
  ```sql
  LOAD DATA LOCAL INFILE '/path/to/file.sql' INTO TABLE tb_user 
  FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n';
  ```

#### 2. 主键优化

- **设计原则**：
  - 在满足业务需求的情况下，尽量降低主键的长度。
  - 尽量选择顺序插入，使用 `AUTO_INCREMENT` 自增主键。
  - 避免使用 UUID 或其他自然主键（如身份证号），以降低索引效率。
  
- **存储与索引维护**：
  - 在 InnoDB 中，数据根据主键顺序组织。
  - 页分裂：页可以为空，也可以填充部分，也可以填充100%，每个页包含了2-N行数据（如果一行数据过大，会行溢出），根据主键排列。
  - 页合并：当删除一行记录时，实际上记录并没有被物理删除，只是记录被标记（flaged）为删除并且它的空间变得允许被其他记录声明使用。当页中删除的记录到达 MERGE_THRESHOLD（默认为页的50%，可自定义，可以在建表，创建索引的时候指定），InnoDB会开始寻找最靠近的页（前后）看看是否可以将这两个页合并以优化空间使用。

#### 3. 排序优化 (ORDER BY)

- **使用索引优化排序**：
  - 使用索引按顺序读取数据，避免 `Using filesort`。
  - 如果排序字段全部为升序或降序，可直接利用索引。
  - 如果有多字段排序（升序与降序），需创建联合索引。
  
- **示例**：
```sql
CREATE INDEX idx_user_age_phone ON tb_user(age ASC, phone DESC);

SELECT id, age, phone 
FROM tb_user 
ORDER BY age ASC, phone DESC;
```

#### 4. 分组优化 (GROUP BY)

- **利用索引**：
  - 在 `GROUP BY` 查询中使用合适的索引，提高效率。
  - 确保索引列满足最左前缀法则。

- **示例**：
```sql
SELECT profession, COUNT(*) FROM tb_user GROUP BY profession ORDER BY profession;
```

#### 5. 分页优化 (LIMIT)

- **优化大数据量分页**：
  - 避免高偏移量的 LIMIT 查询，减少不必要的数据排序。
  
- **使用覆盖索引加速**：
  - 通过主键索引排序查询，可以显著提高性能。
  
- **示例**：
```sql
-- 慢查询示例
SELECT * FROM tb_sku LIMIT 5000000, 10;

-- 优化查询示例
SELECT id FROM tb_sku ORDER BY id LIMIT 5000000, 10;

-- 通过联表查询进行优化
SELECT * 
FROM tb_sku AS s
JOIN (SELECT id FROM tb_sku ORDER BY id LIMIT 5000000, 10) AS a 
ON s.id = a.id;
```

#### 6. 计数优化 (COUNT)

- **性能差异**：
  - `COUNT(*) ≈ COUNT(1)` 性能最好，InnoDB 默认的优化机制。
  - 使用 `COUNT(主键)`、`COUNT(字段)` 的性能依次降低。

- **示例**：
```sql
SELECT COUNT(*) FROM tb_user;                 # 性能最好
SELECT COUNT(1) FROM tb_user; 				  # 和上面差不多
SELECT COUNT(id) FROM tb_user;                # 次优
SELECT COUNT(name) FROM tb_user;              # 低效（可能遍历全表）
```

- **建议**：
  - 尽量使用 `COUNT(*)`，并考虑使用 Redis 或其他方式缓存计数。

#### 7. 更新优化 (UPDATE)

- **避免锁升级**：
  - 确保更新条件(WHERE之后)的字段上有索引，以避免行锁升级为表锁。
  
- **示例**：
```sql
UPDATE student SET no = '042' WHERE id = 1;      -- 行锁(主键索引)
UPDATE student SET no = '114514' WHERE name = 'test'; -- 表锁（name没有索引，需添加索引以优化）
```

-----

### 视图

--------

#### 1. 语法

- 创建视图

  ```sql
  CREATE [ OR REPLACE ]
  VIEW 视图名称[（列名列表）]
  AS 
  SELECT 语句 [ WITH [ CASCADED | LOCAL ] CHECK OPTION ];
  ```

- 显示视图

  ```sql
  SHOW CREATE VIEW [ 视图名称 ]; #显示创建视图语句
  
  SELECT [查询字段] FROM [ 视图名称 ]  WHERE [.....]; #查看创建的视图中的数据
  ```

- 修改

  ```sql
  #方式一
  CREATE [OR REPLACE] 
  VIEW 视图名称[（列名列表)）] 
  AS 
  SELECT 语句[ WITH[ CASCADED | LOCAL ] CHECK OPTION ];
  ---------------------------------------------------------
  #方式二
  ALTER 
  VIEW 视图名称 [（列名列表)] 
  AS 
  SELECT语句 [WITH [CASCADED | LOCAL] CHECK OPTION];
  ```

- 删除

  ```sql
  DROP VIEW [IF EXISTS] 视图名称 [视图名称]
  ```

#### 2. 检查

如果对视图进行插入操作，那么数据将会插入原表之中，如果在创建视图时使用了`WITH CHECK OPTION`字段，那么在进行插入或更新时将会受到创建视图时设置的检查的限制。

两个检查选项：CASCADED 和 LOCAL ，默认值为 CASCADED。

- CASCADED：会检查本视图以及递归检查本视图创建时所依赖的视图设置的限制。
- LOCAL：在`空`的基础上会检查本视图的限制条件，并向上递归检查本视图所依赖的并且设置了检查选项的视图的限制条件
- 空(即不加检查选项)：不会检查本视图的条件，但是会向上检查依赖的并设置了检查选项的视图的限制条件。

#### 3. 更新

要使视图可更新，视图中的行与基础表中的行之间必须存在一对一的关系。如果视图包含以下任何一项，则该视图不可更新

1. 聚合函数或窗口函数 ( SUM()、MIN()、MAX()、COUNT() 等 )
2. DISTINCT
3. GROUP BY
4. HAVING
5. UNION 或者UNION ALL
```sql
#插入失败的例子
REATE stu_v_count AS SELECT COUNT(*) FROM student;

INSERT INTO stu_v_count VALUES(10);
```

作用

- 视图不仅可以简化用户对数据的理解，也可以简化他们的操作。那些被经常使用的查询可以被定义为视图，从而使得用户不必为以后的操作每次指定全部的条件，只需要用户针对于视图进行操作。

- 安全 数据库可以授权，但不能授权到数据库特定行和特定的列上。通过视图用户只能查询和修改他们所能见到的数据，能够屏蔽一些比较敏感的信息，比如密码，身份证号。
- 数据独立，视图可帮助用户屏蔽真实表结构变化带来的影响

- 总而言之 类似于给表加上了一个外壳，通过这个外壳访问表的时候，只能按照所设计的方式进行访问与更新。

------

### 存储过程

简而言之，就是数据库中的函数。

-------

#### 1. 语法

- 创建

  ```sql
  CREATE PROCEDURE 存储过程名称( [参数] ) 
  
  BEGIN
  	 SQL 语句 
  END;
  ```

- 调用

  ```sql
  CALL [存储过程名称](传入参数);
  ```

- 删除

  ```sql
  DROP PROCEDURE [ IFEXISTS ] [存储过程名称];
  ```

  tips: 在命令行中，执行创建存储过程的SQL时，需要通过关键字delimiter来指定SQL语句的结束符。默认是以分号作为结束符。`delimiter $$`，则$$符作为结束符。

  

#### 2. 用户自定义变量

- 定义方法：

  ```sql
  SET @自定义变量 := ?;  # := 和 = 都可以
  
  SELECT @自定义变量 := ?;
  
  SELECT 字段 INTO @自定义变量 FROM ....; #将查询结果传递给自定义变量
  ```

- 在函数中的应用示例：

  ```sql
  CREATE PROCEDURE getcount()
  
  BEGIN
      DECLARE cnt INT DEFAULT 0; #变量声明
      SELECT COUNT(*) INTO cnt FROM goods; #给变量赋值
      SELECT cnt; #输出结果
  END;
  
  CALL getcount();
  ```

#### 3. 存储过程
##### 3.1 IF语句示例：

```sql
CREATE PROCEDURE score()
BEGIN
    DECLARE cnt INT DEFAULT 58;
    DECLARE res VARCHAR(6);
    IF cnt >= 80 THEN		#条件之后接THEN
        SET res := '优秀';
    ELSEIF cnt >= 60 THEN	#注意这里写为ELSEIF 没有空格
        SET res := '及格';
    ELSE					#ELSE 表示最后一个分支
        SET res := '重开算了';
    END IF; #END IF;表示IF语句的结束

    SELECT res;
END;
```

##### 3.2 传参示例：

```sql
CREATE PROCEDURE p1(IN cnt INT, OUT res VARCHAR(10)) #'IN'表示传入参数,'OUT'表示输出
BEGIN												#，'INOUT'表示传入也传出。相当于传入了指针
    IF cnt > 80 THEN
        SET res := '优秀';
    ELSEIF cnt > 60 THEN
        SET res := '及格';
    ELSE
        SET res := '重开算了';
    END IF;
END;

CALL p1(99, @result);
SELECT @result;
```

##### 3.3 CASE语句示例：

```sql
CREATE PROCEDURE p4(IN month INT)
BEGIN
    DECLARE res VARCHAR(25);
    CASE
        WHEN month >= 1 AND month <= 3 THEN
            SET res := '第一季度';
        WHEN month >= 4 AND month <= 6 THEN
            SET res := '第二季度';
        WHEN month >= 7 AND month <= 9 THEN
            SET res := '第三季度';
        WHEN month >= 10 AND month <= 12 THEN
            SET res := '第四季度';
        ELSE
            SET res := '非法参数';
    END CASE;
    SELECT res;
END;

CALL p4(6);
```

##### 3.4 WHILE循环语句示例：

```sql
CREATE PROCEDURE wh(IN n INT)
BEGIN
    WHILE n > 0 DO	#满足条件，则干DO后面的事，END WHILE;表示于一次循环结束.
        SET n = n - 1;
        END WHILE;
    SELECT n;
END;

CALL wh(5);
```

##### 3.5 REPEAT语句示例(相当于do while)

```sql
CREATE PROCEDURE rep(IN n INT)
BEGIN
    DECLARE res INT DEFAULT 0;
    REPEAT
            SET res = res + n;
            SET n = n - 1;
        UNTIL n <= 0
    END REPEAT;
    SELECT res;
END;

CALL rep(5);
```

##### 3.6 LOOP语句示例(相当于无限循环但是带有continue和break的while语句)

```sql
CREATE PROCEDURE l2(IN n INT)
BEGIN
    DECLARE res INT DEFAULT 0;
    sum:LOOP		#sum是给这个循环的一个标签
        IF n <= 0 THEN
            LEAVE sum;  #ITERATE sum 则表示continue，继续该循环，LEAVE sum表示跳出该循环
        END IF;

        SET res = res + n;
        SET n = n - 1;
    END LOOP sum; 	#sum表示sum这个标签的循环语句结束
    SELECT res;
END;

CALL l2(15);
```

##### 3.7 CURSOR游标示例：

```sql
CREATE PROCEDURE p15(IN n DECIMAL(10,2))
BEGIN
    DECLARE gname VARCHAR(255);
    DECLARE gprice DECIMAL(10,2);

    DECLARE cursor_goods CURSOR FOR SELECT price, goods_name FROM goods WHERE price < n; #声明游标
    DECLARE exit HANDLER FOR NOT FOUND CLOSE cursor_goods;	#创建一个退出条件并设置退出时执行的语句(关闭游标)

    DROP TABLE IF EXISTS p12_test;
    CREATE TABLE p12_test(
        id INT PRIMARY KEY AUTO_INCREMENT,
        price DECIMAL(10,2),
        goods_name VARCHAR(255)
    );

    OPEN cursor_goods;#打开游标后循环遍历所有的数据
    WHILE TRUE DO
        FETCH cursor_goods INTO gprice, gname;
        INSERT INTO p12_test(price, goods_name) VALUES(gprice, gname);
        END WHILE;
END;

CALL p15(51.15);
```

##### 3.8 条件处理程序

  - 在 MySQL 存储过程中，条件处理程序用于处理运行时可能遇到的特定条件，如警告、未找到结果或其他 SQL 错误。根据不同的条件，可以选择继续执行程序、终止当前程序或执行其他清理操作。

  -  语法

    ```sql
    DECLARE handler_action HANDLER FOR condition_value
        [statement]
    ```

  - **handler_action**：可以是 `CONTINUE` 或 `EXIT`。
    - `CONTINUE`：在遇到指定条件时，继续执行后续程序。
    - `EXIT`：在遇到指定条件时，终止当前程序的执行。


  - **condition_value**：指定的 SQL 条件，例如：
    - `SQLSTATE` 值，例如 `02000`（表示没有数据行被返回）。
    - `SQLWARNING`：所有以 01 开头的 SQLSTATE 代码的简写。
    - `NOT FOUND`：所有以 02 开头的 SQLSTATE 代码的简写。
    - `SQLEXCEPTION`：捕获所有未被 `SQLWARNING` 或 `NOT FOUND` 捕获的 SQLSTATE 代码。
  - 执行示例可以看游标的示例语句

##### 3.9 有返回值的函数

```sql
CREATE FUNCTION ad(n INT)
RETURNS INT DETERMINISTIC	#定义返回值类型
BEGIN
    DECLARE res INT DEFAULT 0;

    WHILE n >= 0 DO
        SET res = res + n;
        SET n = n - 1;
    END WHILE;

    RETURN res;
END;

SELECT ad(15);
```

------

### 触发器

-----

#### 1. 基本语法

```sql
CREATE TRIGGER trigger_name
[ BEFORE | AFTER ] [ INSERT | UPDATE | DELETE ]
ON table_name
FOR EACH ROW
BEGIN
    -- 触发器逻辑
END;
```

#### 2. 作用

触发器（Trigger）用于在对表进行插入（INSERT）、更新（UPDATE）或删除（DELETE）操作时，自动执行某一特定操作。触发器可以帮助我们维持数据完整性、自动记录审计日志、实现复杂的业务逻辑等。

#### 3. 注意事项

- 触发器会在每一行影响的情况下执行，因此在高并发或大批量数据处理时会对性能有影响。
- 触发器不能被直接调用，并且不能修改调用触发器的表。
- 在设计触发器时，要注意避免在触发器内进行无限递归调用。

-----

