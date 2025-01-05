# AOP (Aspect-Oriented Programming)

## 簡介
AOP（面向切面編程，Aspect-Oriented Programming）是一種編程範式，用於在不修改核心業務邏輯的情況下，透過切面（Aspect）動態地向程式中添加橫切關注點（Cross-Cutting Concerns） ，如日誌記錄、安全檢查、事務管理等。

## 核心概念
+ 切面（Aspect）: 切面是AOP的核心模組，用來定義橫切關注點的功能。
例如：記錄日誌、效能監控。

+ 連接點（Join Point）
程式執行的某個特定點，例如方法呼叫、異常拋出等。
例如：在方法執行前或執行後。

+ 切入點（Pointcut）
定義在程式中選擇的一個或多個連接點，切面程式碼會套用到這些點。
例如：選擇所有public方法作為切入點。

+ 通知（Advice）
在特定的切入點上執行的程式碼邏輯。
通知類型包括：
    + 前置通知（Before Advice）：在方法執行前執行。
    + 後置通知（After Advice）：在方法執行後執行。
    + 環繞通知（Around Advice）：包裹在方法執行的前後。
    + 異常通知（After Throwing Advice）：在方法拋出異常後執行。
    + 返回通知（After Returning Advice）：在方法成功返回後執行。

+ 織入（Weaving）
將切面程式碼應用到目標物件上的過程。
織入可以在以下階段完成：
    + 編譯時織入：在編譯階段完成，例如AspectJ。
    + 運行時織入：在運行階段動態代理，例如Spring AOP。
    + 類別載入時織入：在字節碼被類別載入器載入時完成。

## 應用場景
+ 日誌記錄
+ 安全性檢查
+ 事務管理
+ 效能監控
+ 異常處理
