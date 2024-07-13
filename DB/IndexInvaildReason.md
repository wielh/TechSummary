# 索引失效

## 索引失效可能原因

+ 使用 != 、 <>、NOT 查詢可能會變全掃

```
SELECT * FROM user WHERE age != 20 ( bad 全掃 )
SELECT * FROM user WHERE age +1 <> 20 ( bad 全掃 )
SELECT * FROM user WHERE age NOT IN(20) ( bad 全掃 )
```

+ 用 like 且 % 在前面有索引也會變全掃

```
SELECT * FROM user WHERE name like '%-Mark' ( bad 全掃 )
```

+ 誤用 or 時會變全掃
    + 索引欄位: {age}
    ```
    SELECT * FROM user WHERE age = 18 OR name = 'C-Ian'; ( bad 全掃 )
    ```

    + 索引欄位: {age},{name} 
    ```
    SELECT * FROM user WHERE age = 18 OR name = 'C-Ian'; ( good 索引 )
    ```

    + 索引欄位: {age}
    ```
    SELECT * FROM user WHERE age = 18 AND name = 'C-Ian'; ( good 索引 )
    ```


+  在 WHERE 欄位進行運算
    + 索引欄位: {age}
    ```
        SELECT * FROM user WHERE age/2 = 18; ( bad 全掃 )
        SELECT * FROM user WHERE age = 18*2; ( good 索引 )
    ```

+  使用一些函數
   ```
    SELECT * FROM test.user where age >= RAND(); ( bad 全掃 )
   ```

+  对索引隐式类型转换

+  联合索引非最左匹配

    Note. 有一个比较特殊的查询条件：where a = 1 and c = 3 ，符合最左匹配吗？
    
    MySQL 5.5 的话，前面 a 会走索引，在联合索引找到主键值后，开始回表，到主键索引读取数据行，Server 层从存储引擎层获取到数据行后，然后在 Server 层再比对 c 字段的值。从 MySQL 5.6 之后，有一个索引下推功能，可以在存储引擎层进行索引遍历过程中，对索引中包含的字段先做判断，直接过滤掉不满足条件的记录，再返还给 Server 层，从而减少回表次数。

    索引下推的大概原理是：截断的字段不会在 Server 层进行条件判断，而是会被下推到「存储引擎层」进行条件判断（因为 c 字段的值是在 (a, b, c) 联合索引里的），然后过滤出符合条件的数据后再返回给 Server 层。由于在引擎层就过滤掉大量的数据，无需再回表读取数据来进行判断，减少回表次数，从而提升了性能。
    