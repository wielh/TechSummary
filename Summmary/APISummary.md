# 開發 API 流程

## 技術選型

技術選型包含作業系統，DB，後端程式語言與中間件。

+ 作業系統: Windows, Linux; 廣義一點 Docker, AWS 或是本地伺服器也算。

+ DB: MySQL, MongoDB, Redis,....

+ 程式語言: Java(spring), python(flask), go(beego, gin),....

## 架構上需要考慮的事

+ DB 架構

    + Indexing: 根據查詢需要，在DB建立合適的 Index。

    + 緩存: 針對熱點數據和需要即時計算與更新的數據，可以增加緩存。緩存數據需要考量是放在redis或API程序的變數裡。

    + 一致性: 根據場景，對可用性與一致性進行取捨，比如DB的隔離等級。考慮到效率，盡量避免在DB層面檢查數據限制，而是交由API管理。

+ API 主體架構
    
    + 登入: 使用 session/token/Oauth? 框架有沒有現成的登入實作?

    + 單例/微服務集群: 詳細比較可以到[此連接](../General/MircoService.md)觀看

    + ORM: 使用語言相關的ORM框架，ORM框架至少有以下功能: 可以防止SQL injection、SQL連接池管理、SQL語句預處理、與方便遷移。

    + Config管理:  可以用文件(json,yaml,...)進行統一管理; DB密碼或證書之類的機密考慮用系統變數儲存。如果類似微服務集群那樣的各微服務IP經常變動，可以考慮使用註冊中心。

    + 常見中間件: 登入驗證(token/session/oauth2)，限流，cors，crsf，logger，timeout，error handle, static, requestID

+ 運行效率

    + 多執行緒處理: 通常適合 CPU 密集型任務，或是許多類似且獨立的任務執行。

    + 異步處理: 通常適合 I/O 密集型任務，特別是調用DB或第三方API。

    + 批量處理: 一次提交許多命令給程式執行，會一筆筆提交要好，因為這樣可以節省物件(網路連線，DB連線，線程)創建與銷毀的成本

+ 程序穩定

    + 通用錯誤處理: 框架中有沒有 filter/handler 之類的方法處理剩下未處理的error，防止因意外而造成程式崩潰。

    + 多執行緒如果有修改到共用數據的話，必須確認是否引發竞争条件(race condition)。

    + 根據語言特性不同，有時需要特別注意 Object 為 null 時會不會引發崩潰。

    + 檢查極短時間同時有多個同樣的請求提交，API是否按照預期執行。

    + 日誌: 有沒有框架、工具快速紀錄、查詢程序運行錯誤。

    + 消息中間件: 是一種用於在分散式系統中實現非同步通訊的技術，可用於

        + 組件之間的解耦

        + 最大流量限制

        + 程式異步執行

        + 數據平均分發，以達到負載平衡

+ 資料安全

    + 需要記錄登入資訊(username, IP, deviceID, ...)，並對登入次數進行限制，失敗次數過多對用戶通知。

    + 避免直接以明文的方式儲存密碼，身分證字號等機密訊息

    + 避免打印敏感資料，如果可以的話也避免儲存敏感資料

    + 需要嚴格檢查取得的資料是否為該用戶所有。比如 user A 只允許查閱 A 相關的資料。
    
    + 避免使用常見的Hash方法，要為不同的使用者加入不同的Salt

    + 需要使用 Crypto random generator，並避免使用 time.Now() 做為亂數種子

+ 測試 

    + Functional Testing: 
        + unit test: 確保元件功能正常。
        + integration test: 確保 API 功能按照設計要求正確運作。

    + Perforence Testing: 
        + 負載測試（Load Testing）: 驗證 API 在同時使用者或要求增加時的效能表現。
        + 壓力測試（Stress Testing）: 測試 API 在超出設計容量的情況下的行為。
        + 容量測試（Capacity Testing）: 決定 API 的最大處理能力。

    + Security Testing:
        + 使用無效的 Token 存取需要認證的 API，驗證是否拒絕存取。
        + 嘗試注入惡意 SQL，檢查 API 是否能夠防禦。
        
    + Error Handling Testing:
        + 测试超时、连接失败等异常是否能被正确捕获
        + 模拟数据库/第三方API断开连接，验证 API 是否返回合适的错误信息
        + 如果輸入不符合格式的數據，確保處理妥當並返回預期錯誤訊息。

    + CI/CD: 可以將以上流程寫進自動化測試，提交code時自動測試，通過才可接受commit。
     