# Redis 分佈式鎖設計方案

在分散式系統中，傳統的語言級鎖（如 Go 的 `sync.Mutex`）只能限制單個進程內的併發。為了跨多個節點（Server）協調資源，我們需要使用 **Redis 分佈式鎖**。

以下是四種常見的 Redis 鎖設計方案與實作細節：

---

## 1. 基礎方案：單節點分佈式鎖

這是最常用的非重入鎖設計，適合絕大多數簡單的併發排他場景。

### A. 獲取鎖 (Acquire)
使用原子性的 `SET NX EX` 指令。
```sql
SET lock_key unique_value NX EX 30
```
* **`NX`**：只有當 `lock_key` 不存在時才寫入成功（保證互斥性）。
* **`EX 30`**：設定 30 秒過期時間（防範進程宕機導致鎖無法釋放，造成死鎖）。
* **`unique_value`**：**必須是每個線程唯一的隨機值**（例如 UUID），用以標記鎖的持有者。

### B. 釋放鎖 (Release)
釋放鎖時必須使用 **Lua 腳本** 進行「比對與刪除」的原子操作。防止交易 A 因為執行時間過長導致鎖過期，此時交易 B 獲取了鎖，而交易 A 執行完後誤刪了交易 B 的鎖。

```lua
-- KEYS[1]: 鎖的 Key
-- ARGV[1]: 線程持有的隨機 Value (unique_value)
if redis.call("GET", KEYS[1]) == ARGV[1] then
    return redis.call("DEL", KEYS[1])
else
    return 0
end
```

---

## 2. 進階方案：基於 Hash 的可重入鎖 (Reentrant Lock)

**可重入鎖**允許同一個線程在未釋放鎖的情況下，多次重複獲取同一把鎖。這在遞迴調用或複雜巢狀業務中非常有用。

### A. 實作原理
Redis 使用 **Hash** 結構來儲存鎖：
* **`Key`**：鎖的名稱。
* **`Field`**：線程的唯一識別（例如 `UUID:ThreadID`）。
* **`Value`**：重入次數（計數器，每次獲取 +1，釋放 -1）。

### B. 獲取鎖 Lua 腳本
```lua
-- KEYS[1] = 鎖名稱 (Lock Name)
-- ARGV[1] = 鎖過期時間 (毫秒)
-- ARGV[2] = 線程標識 (UUID:ThreadID)

-- 1. 如果鎖不存在，直接建立並設計數為 1
if redis.call('exists', KEYS[1]) == 0 then
    redis.call('hset', KEYS[1], ARGV[2], 1)
    redis.call('pexpire', KEYS[1], ARGV[1])
    return 1
end

-- 2. 如果鎖已存在，且持有者是自己，計數器 +1 并刷新過期時間
if redis.call('hexists', KEYS[1], ARGV[2]) == 1 then
    redis.call('hincrby', KEYS[1], ARGV[2], 1)
    redis.call('pexpire', KEYS[1], ARGV[1])
    return 1
end

-- 3. 鎖被其他人持有，獲取失敗
return 0
```

### C. 釋放鎖 Lua 腳本
```lua
-- KEYS[1] = 鎖名稱
-- ARGV[1] = 鎖過期重設時間 (毫秒)
-- ARGV[2] = 線程標識 (UUID:ThreadID)

-- 1. 如果自己不是鎖的持有者，無權釋放，返回 nil
if redis.call('hexists', KEYS[1], ARGV[2]) == 0 then
    return nil
end

-- 2. 計數器 -1
local counter = redis.call('hincrby', KEYS[1], ARGV[2], -1)

-- 3. 判斷是否完全釋放
if counter > 0 then
    -- 仍有重入，更新過期時間，鎖依然有效
    redis.call('pexpire', KEYS[1], ARGV[1])
    return 0
else
    -- 計數歸零，完全釋放鎖，刪除 Key
    redis.call('del', KEYS[1])
    return 1
end
```

---

## 3. 公平方案：基於 ZSet 的隊列鎖 (Fair Lock)

預設的分佈式鎖是非公平的，多個線程競爭時完全憑隨機和網路延遲。如果業務要求**先到先得（先進先出 FIFO）**，則需要引入隊列鎖。

### A. 實作原理
1. 使用 **ZSet (有序集合)** 作為排隊佇列，**Score 設定為請求鎖時的時間戳**，Member 為 `ThreadID`。
2. 當線程嘗試獲取鎖時，先將自己加入 ZSet 佇列中。
3. 檢查 ZSet 的排在第一位（Score 最小）的成員是否是自己：
   * 如果是，代表輪到自己，成功獲取鎖，並將自己從 ZSet 移出。
   * 如果不是，代表前方有人排隊，線程進入等待或重試。

---

## 4. 效能優化方案：基於 Pub/Sub 的通知鎖

### A. 傳統輪詢鎖的效能問題
在常見的 Go 或 Java 實作中，當獲取鎖失敗時，代碼通常會寫成：
```go
for {
    if acquireLock() { break }
    time.Sleep(100 * time.Millisecond) // 輪詢等待
}
```
這種**忙輪詢 (Busy-polling)** 會對 Redis 伺服器造成無謂的查詢壓力，且 `time.Sleep` 使得鎖釋放後，等待中的線程無法「即時」得到鎖，造成效能浪費。

### B. 解決方案：發布與訂閱 (Publish/Subscribe)
1. 當線程 A 獲取鎖失敗時，**訂閱一個與該鎖關聯的頻道**（例如 `lock_release_channel:lock_name`）。
2. 線程 A 進入阻塞等待（Go 中可利用 channel 阻塞，Java 可用 Semaphore 或 Future）。
3. 當持有鎖的線程 B 釋放鎖時，除了刪除 Redis Key，還向該頻道發送一條消息：`PUBLISH lock_release_channel:lock_name "released"`。
4. 線程 A 收到訂閱消息後，**被喚醒並立刻嘗試獲取鎖**。
5. 這樣可以免除無謂的輪詢，將 CPU 與 Redis I/O 消耗降到最低。
