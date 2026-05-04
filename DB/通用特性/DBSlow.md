# mysql 指令應避免事項

## 索引失效原因

+ where的index覆蓋率低或查詢沒用到index

+ 使用 != 、 <>、NOT 查詢可能會變全掃

```sql
SELECT * FROM user WHERE age != 20 ( bad 全掃 )
SELECT * FROM user WHERE age +1 <> 20 ( bad 全掃 )
SELECT * FROM user WHERE age NOT IN(20) ( bad 全掃 )
```

+ 用 like 且 % 在前面有索引也會變全掃

```sql
SELECT * FROM user WHERE name like '%-Mark' ( bad 全掃 )
```

+ 誤用 or 時會變全掃
  + 索引欄位: {age}

    ```sql
    SELECT * FROM user WHERE age = 18 OR name = 'C-Ian'; ( bad 全掃 )
    ```

  + 索引欄位: {age},{name}

    ```sql
    SELECT * FROM user WHERE age = 18 OR name = 'C-Ian'; ( good 索引 )
    ```

  + 索引欄位: {age}

    ```sql
    SELECT * FROM user WHERE age = 18 AND name = 'C-Ian'; ( good 索引 )
    ```

+ 在 WHERE 欄位進行運算
  + 索引欄位: {age}

    ```sql
    SELECT * FROM user WHERE age/2 = 18; ( bad 全掃 )
    SELECT * FROM user WHERE age = 18*2; ( good 索引 )
    ```

+ 使用一些函數

   ```sql
    SELECT * FROM test.user where age >= RAND(); ( bad 全掃 )
   ```

+ 對索引隱式型別轉換

+ 聯合索引非最左匹配

    Note. 有一個比較特殊的查詢條件：where a = 1 and c = 3 ，符合最左匹配嗎？

    MySQL 5.5 的話，前面 a 會走索引，在聯合索引找到主鍵值後，開始回表，到主鍵索引讀取數據行，Server 層從存儲引擎層獲取到數據行後，然後在 Server 層再比對 c 欄位的值。從 MySQL 5.6 之後，有一個索引下推功能，可以在存儲引擎層進行索引遍歷過程中，對索引中包含的欄位先做判斷，直接過濾掉不滿足條件的記錄，再返還給 Server 層，從而減少回表次數。

    索引下推的大概原理是：截斷的欄位不會在 Server 層進行條件判斷，而是會被下推到「存儲引擎層」進行條件判斷（因為 c 欄位的值是在 (a, b, c) 聯合索引裡的），然後過濾出符合條件的數據後再返回給 Server 層。由於在引擎層就過濾掉大量的數據，無需再回表讀取數據來進行判斷，減少回表次數，從而提升了性能。

## 全表鎖定

+ 觸發時機
  + DDL 操作:ALTER ,RENAME ,DROP ,CREATE INDEX
  + MyISAM 引擎
  + 優化查詢條件沒有用上索引
