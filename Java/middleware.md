# Springboot Middlwware

## 簡介

在 Spring Boot 中，如果你在開發 Web 應用（特別是使用 Spring MVC），middleware 可能指的是：
+ Filter（javax.servlet.Filter）
+ Interceptor（HandlerInterceptor）

## Filter 特點
+ 可攔截的請求: 所有請求（靜態資源也可）
+ 適用場景: 安全性、日誌記錄、CORS、編碼等前處理
+ 執行順序: 由容器管理，先於 Spring MVC

## Interceptor 特點
+ 可攔截的請求: 控制器請求（Controller 處理前後）
+ 適用場景: 權限控制、日誌、資料預處理、封裝參數等
+ 執行順序: 在 Controller 執行前/後

