# API 通訊協定對比 (REST vs. gRPC vs. GraphQL)

在設計系統架構或進行前後端協作時，選擇正確的 API 通訊協定對系統的效能、擴展性與開發體驗有著決定性的影響。目前主流的三種技術為 **REST**、**gRPC** 和 **GraphQL**。

---

## 1. REST (Representational State Transfer)

REST 是目前 Web 開發中最普遍的架構風格，以「**資源 (Resource)**」為核心。

* **運作機制**：
  * 使用標準 HTTP 方法（`GET`, `POST`, `PUT`, `DELETE`）對資源進行操作。
  * 數據格式通常採用 **JSON**（純文字，易讀）。
  * 遵循無狀態（Stateless）與統一接口（Uniform Interface）原則。
* **優點**：
  * 簡單直觀，開發門檻低，瀏覽器天然原生支援。
  * HTTP 基礎設施完整（如快取控制 `Cache-Control` 支援佳）。
  * 鬆耦合，適用於對外開放的公共 API。
* **缺點（痛點）**：
  * **過度拉取 (Over-fetching)**：回傳的 JSON 包含客戶端不需要的冗餘欄位。
  * **拉取不足 (Under-fetching)**：為了渲染一個畫面，客戶端必須發送多次請求（例如先拿 Order，再用 OrderID 拿 Items）。
  * 無內建的強型別約束，文檔（如 Swagger/OpenAPI）需要額外維護，容易過期。

---

## 2. gRPC (Google Remote Procedure Call)

gRPC 是由 Google 開發的高效能、開源的遠端程序呼叫 (RPC) 框架，專為**微服務內網通訊 (Inter-service Communication)** 而生。

* **運作機制**：
  * 基於 **HTTP/2** 協議，支援多路複用 (Multiplexing)、標頭壓縮與雙向串流。
  * 使用 **Protocol Buffers (Protobuf)** 作為介面定義語言 (IDL) 與序列化格式，將數據編碼為高效的**二進位格式**進行傳輸。
* **通訊模式**：
  1. **單向 (Unary)**：傳統的 Request-Response。
  2. **伺服器串流 (Server Streaming)**：一請求，多回應（如即時數據推送）。
  3. **客戶端串流 (Client Streaming)**：多請求，一回應（如大檔案上傳）。
  4. **雙向串流 (Bidirectional Streaming)**：雙方可隨時發送與接收（如即時聊天）。
* **優點**：
  * **極高極快**：二進位壓縮與 HTTP/2 使得傳輸體積小、速度快、CPU 消耗低。
  * **強型別契約**：在 `.proto` 檔案中定義介面，可自動生成 Go, Java, Python, TS 等多種語言的客戶端與服務端代碼。
* **缺點**：
  * 瀏覽器無法直接原生調用（需透過 gRPC-Web 或代理 Gateway 轉換）。
  * 數據為二進位，不具備人類可讀性，除錯較為繁瑣。
  * Schema 變更較為嚴格。

---

## 3. GraphQL

GraphQL 是由 Facebook 開發的資料查詢語言，將資料查詢的主導權移交給**客戶端**。

* **運作機制**：
  * 單一端點（通常是 `POST /graphql`）。
  * 客戶端在請求中發送「查詢語句 (Query)」，精確描述所需要的欄位，伺服器則回傳符合該結構的 JSON。
* **優點**：
  * **解決 Over/Under-fetching**：客戶端要什麼給什麼，一次請求搞定關聯資料，非常適合複雜的前端/行動端畫面。
  * **自帶文檔與強型別**：透過 Schema 定義，前端可直接利用工具查詢支援的欄位與型別。
* **缺點**：
  * **後端複雜度高**：容易面臨 N+1 查詢問題（後端需要使用 `Dataloader` 機制進行批次合併查詢）。
  * **快取困難**：因為所有請求都發送到同一個端點且多使用 `POST` 方法，難以利用傳統的 HTTP 代理快取，需依賴客戶端快取（如 Apollo Client）。
  * 查詢語句如果過於複雜（巢狀過深），可能導致伺服器端解析超載。

---

## 4. 三者綜合對比

| 特性 | REST | gRPC | GraphQL |
| :--- | :--- | :--- | :--- |
| **通訊範式** | 資源導向 (Resource) | 方法導向 (Procedure) | 查詢導向 (Query) |
| **傳輸協議** | HTTP/1.1 或 HTTP/2 | **HTTP/2 (強制)** | HTTP/1.1 或 HTTP/2 |
| **序列化格式** | JSON / XML (純文字) | **Protobuf (二進位)** | JSON (純文字) |
| **型別安全** | 弱（需外掛 OpenAPI） | **強 (編譯期自動生成)** | 強 (Schema 定義) |
| **數據精準性** | 差 (存在 Over/Under fetching) | 差 (由後端 Struct 決定) | **極佳 (由客戶端宣告式拉取)** |
| **通訊模式** | Request-Response | Unary, Streaming | Request-Response, Subscription |
| **適用場景** | 公共 API、微服務對外閘道 | **微服務內部通訊、IoT、即時串流** | **複雜的前後端互動、BFF (Backend For Frontend)** |
