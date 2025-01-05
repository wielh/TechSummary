# DB 框架關係

## 總結
+ JDBC：底層的資料庫操作 API，開發者需要手動處理 SQL 和資料庫連接。
+ JPA：Java 標準的 ORM API，使開發者不必手動編寫 SQL，並提供了對象與資料庫的映射。
+ Hibernate：JPA 的實現，提供了更多的功能和擴展，並且支持高效的查詢和緩存等。

## JDBC (Java Database Connectivity)
JDBC 是 Java 提供的一個標準 API，用於與關聯型資料庫進行互動。它允許開發者使用 SQL 查詢來操作資料庫。
JDBC 的主要特點：
+ 直接與資料庫進行交互。
+ 需要開發者手動處理 SQL 查詢語句和資料的映射。
+ 提供了低級的操作方式，開發者需要編寫 SQL 語句來進行 CRUD（Create、Read、Update、Delete）操作。
+ 需要管理連接、結果集、事務等。

## JPA (Java Persistence API)
JPA 是 Java 提供的標準持久化 API，用於簡化 Java 應用程式中的資料庫操作。JPA 是一個高級的 API，相對於 JDBC 它提供了對象關聯映射（ORM）功能，將資料庫中的資料轉換為 Java 對象，並且幫助開發者處理一些繁瑣的底層操作，如資料的持久化、查詢、更新和刪除。

+ 基於對象關聯映射（ORM）的思想，將資料庫表映射成 Java 類，並將資料庫中的記錄映射成 Java 對象。
+ 提供了標準化的 API，開發者不需要關心具體的資料庫操作細節。
+ 可以與多種不同的 JPA 實現（如 Hibernate、EclipseLink、OpenJPA 等）一起使用。

## Hibernate
Hibernate 是一個流行的開源框架，實現了 JPA 介面，並提供了豐富的功能來處理對象關聯映射（ORM）。它是 JPA 的一種實現，並且可以與 JPA 一起使用，但 Hibernate 提供了比 JPA 更多的功能和擴展。

+ 提供了與 JPA 相同的功能，但擴展了更多自定義選項（例如，緩存、批量處理、更多的查詢方式等）。
+ 支持 HQL（Hibernate Query Language），類似於 JPQL，但功能上更強大。
+ 提供了更強大的功能，比如二級緩存、自動生成 SQL 等。

Hibernate 是 JPA 的一個實現，符合 JPA 規範。
你可以將 Hibernate 視為 JPA 的具體實現，並可以選擇使用 Hibernate 擴展的功能。
JPA 和 Hibernate 的比較：
JPA 是一個規範，而 Hibernate 是 JPA 的一個實現。JPA 定義了如何進行對象關聯映射（ORM）操作，而 Hibernate 提供了一些更高級的功能來支持 JPA。
JPA 提供了對應的接口和抽象，開發者通常不需要關心底層的 ORM 實現（如 Hibernate），只需關注 JPA 提供的標準 API。
Hibernate 除了支持 JPA 規範外，還提供了額外的功能，比如查詢語言 HQL、更高效的緩存機制等。如果你需要這些額外功能，可以直接使用 Hibernate。
