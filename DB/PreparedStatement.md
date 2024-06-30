# PreparedStatement(預編譯語句)

## 簡介
Prepared Statements 是一种在数据库操作中使用的技术，可以预编译 SQL 语句，提高执行效率，并增强安全性，防止 SQL 注入攻击。

## Java Example
```
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class Example {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/mydatabase";
        String user = "username";
        String password = "password";

        try (Connection conn = DriverManager.getConnection(url, user, password)) {
            String sql = "INSERT INTO users (name, email) VALUES (?, ?)";
            PreparedStatement pstmt = conn.prepareStatement(sql);
            pstmt.setString(1, "John Doe");
            pstmt.setString(2, "john.doe@example.com");

            pstmt.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}

```

## Node.js Example
```
var sql = require('mssql');
var config = {
    server: 'localhost',
    database: 'MyDB',
    user: 'MyUser',
    password: '*******',
    port: '1433'
};

export async function newItem(someValue1: string, someValue2: string) {
    let db;
    try {
      db = await new sql.ConnectionPool(config).connect();
    }
    catch(err) {
      console.error("Connection failed: " + err);
      throw(err);
    }
    try {     
      console.info("Connected")
      const insertStatement = 'INSERT INTO [dbo].[MyData]([SomeValue1],[SomeValue2])VALUES(@SomeValue1, @SomeValue2)';
      const ps = new sql.PreparedStatement(db);
      ps.input('SomeValue1', sql.String)
      ps.input('SomeValue2', sql.String)
      await ps.prepare(insertStatement);
      await ps.execute({
            SomeValue1: someValue1,
            SomeValue2: someValue2
        });
      await ps.unprepare();
    } catch (e) {
        console.error(e);
        throw(e);
    }
}
```

## Go Example
```
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/go-sql-driver/mysql"
)

func main() {
    dsn := "username:password@tcp(localhost:3306)/mydatabase"
    db, err := sql.Open("mysql", dsn)
    if err != nil {
        fmt.Println("Error connecting to the database:", err)
        return
    }
    defer db.Close()

    stmt, err := db.Prepare("INSERT INTO users (name, email) VALUES (?, ?)")
    if err != nil {
        fmt.Println("Error preparing statement:", err)
        return
    }
    defer stmt.Close()

    res, err := stmt.Exec("John Doe", "john.doe@example.com")
    if err != nil {
        fmt.Println("Error executing statement:", err)
        return
    }

    id, err := res.LastInsertId()
    if err != nil {
        fmt.Println("Error getting last insert ID:", err)
        return
    }

    fmt.Println("Inserted Row ID:", id)
}
```