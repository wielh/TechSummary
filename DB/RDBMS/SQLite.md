# SQLite 特點

## 特性

SQLite 是一個輕量級、嵌入式的關聯式資料庫引擎。它被廣泛使用於桌面應用、手機應用、嵌入式系統以及輕量的伺服器應用程式中。以下是一些關於 SQLite 的重要特性和說明：

SQLite 的特性：

+ 嵌入式資料庫：SQLite 是一個內嵌式資料庫，意味著它不需要單獨的伺服器進程。應用程式直接將它嵌入到自己的進程中，這使得 SQLite 非常適合小型或單用戶應用程式。

+ 單一檔案資料庫：所有的資料、模式（schema）、索引（index）、以及交易日誌都存儲在一個單一的檔案中。

+ 無需設定：SQLite 不需要安裝或管理資料庫伺服器，無需額外的設定。

+ 全自動化：SQLite 是自動管理的，不需要專門的資料庫管理員（DBA）。應用程式可以自動創建、打開、關閉、或刪除資料庫。

+ 事務支援：SQLite 完全支援 ACID（Atomicity, Consistency, Isolation, Durability）屬性的事務處理，確保資料完整性和一致性。

+ 並發性支援：SQLite 支援多個讀操作同時進行，但寫操作需要排隊。SQLite 通過鎖機制來管理並發讀寫，雖然不適合高併發的寫操作，但在小型應用程式中表現優異。

+ 小型佔用：SQLite 資料庫引擎的代碼很小，只有幾百 KB，非常適合嵌入到應用程式中。

## SQLite 的應用場景

+ 桌面應用程式：如網頁瀏覽器、圖形編輯工具等。

+ 移動應用程式：iOS 和 Android 應用經常使用 SQLite 作為內部的資料庫引擎。

+ 嵌入式系統：如車載導航、機頂盒、相機、家電設備等。

+ 測試和原型開發：由於其無需設定和易用性，SQLite 是快速測試和原型開發的理想選擇。

+ 網頁應用中的本地儲存：許多現代瀏覽器內建了 SQLite 作為 Web SQL API 的實現，儲存本地資料。