# 如何使用 Redis (Go 語言範例)

本篇文件提供基於官方推薦的 Go 用戶端 **`go-redis/v9`** 對 Redis 主要資料型態進行操作的程式碼範例。

---

## 1. 初始化連接

首先，確保安裝了最新版的 `go-redis` 套件：
```bash
go get github.com/redis/go-redis/v9
```

在 Go 中初始化 `redis.Client` 的範例：

```go
package main

import (
	"context"
	"fmt"
	"log"
	"time"

	"github.com/redis/go-redis/v9"
)

var ctx = context.Background()

func initClient() *redis.Client {
	rdb := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "", // 若無密碼則留空
		DB:       0,  // 使用預設的 DB 0
	})

	// 透過 Ping 測試連接是否正常
	_, err := rdb.Ping(ctx).Result()
	if err != nil {
		log.Fatalf("無法連接至 Redis: %v", err)
	}
	return rdb
}
```

---

## 2. String (字串) 操作

String 是最基礎且常用的型態，適用於快取、計數器與分散式鎖。

```go
func stringExample(rdb *redis.Client) {
	key := "user:100:name"

	// ====== SET (寫入) ======
	// 設定過期時間為 10 分鐘
	err := rdb.Set(ctx, key, "Bob", 10*time.Minute).Err()
	if err != nil {
		log.Fatalf("Set 失敗: %v", err)
	}

	// ====== GET (讀取) ======
	val, err := rdb.Get(ctx, key).Result()
	if err == redis.Nil {
		fmt.Println("Key 不存在")
	} else if err != nil {
		log.Fatalf("Get 失敗: %v", err)
	} else {
		fmt.Printf("取得 user 100 名稱: %s\n", val)
	}

	// ====== INCR / DECR (原子計數器) ======
	counterKey := "page:views"
	// 每次呼叫 +1
	newVal, err := rdb.Incr(ctx, counterKey).Result()
	if err != nil {
		log.Fatalf("Incr 失敗: %v", err)
	}
	fmt.Printf("目前頁面瀏覽次數: %d\n", newVal)
}
```

---

## 3. List (列表) 操作

List 是雙向鏈結串列，常用於簡單的佇列或最新事件紀錄。

```go
func listExample(rdb *redis.Client) {
	listKey := "tasks:queue"

	// ====== PUSH (從右側推入資料) ======
	err := rdb.RPush(ctx, listKey, "task1", "task2", "task3").Err()
	if err != nil {
		log.Fatalf("RPush 失敗: %v", err)
	}

	// ====== RANGE (獲取區間內所有元素) ======
	// 0 到 -1 代表獲取全部
	values, err := rdb.LRange(ctx, listKey, 0, -1).Result()
	if err != nil {
		log.Fatalf("LRange 失敗: %v", err)
	}
	fmt.Printf("當前佇列內的所有任務: %v\n", values)

	// ====== POP (從左側彈出並處理任務) ======
	task, err := rdb.LPop(ctx, listKey).Result()
	if err != nil {
		log.Fatalf("LPop 失敗: %v", err)
	}
	fmt.Printf("處理彈出任務: %s\n", task)

	// 清理 Key
	rdb.Del(ctx, listKey)
}
```

---

## 4. Hash (哈希) 操作

Hash 特別適合儲存結構化物件（如 User Profile），可以針對單一欄位進行更新而不用重寫整個物件。

```go
func hashExample(rdb *redis.Client) {
	hashKey := "user:100:profile"

	// ====== HSET (寫入多個鍵值對) ======
	err := rdb.HSet(ctx, hashKey, map[string]interface{}{
		"name":  "Alice",
		"age":   30,
		"email": "alice@example.com",
	}).Err()
	if err != nil {
		log.Fatalf("HSet 失敗: %v", err)
	}

	// ====== HGET (獲取單一欄位) ======
	name, err := rdb.HGet(ctx, hashKey, "name").Result()
	if err != nil {
		log.Fatalf("HGet 失敗: %v", err)
	}
	fmt.Printf("姓名: %s\n", name)

	// ====== HGETALL (獲取全部物件資料) ======
	profile, err := rdb.HGetAll(ctx, hashKey).Result()
	if err != nil {
		log.Fatalf("HGetAll 失敗: %v", err)
	}
	fmt.Println("完整用戶資料:", profile)

	// ====== HDEL (刪除特定欄位) ======
	err = rdb.HDel(ctx, hashKey, "email").Err()
	if err != nil {
		log.Fatalf("HDel 失敗: %v", err)
	}

	// 清理整個 Key
	rdb.Del(ctx, hashKey)
}
```

---

## 5. ZSet (有序集合) 操作

ZSet 中的每個元素都有一個 Score（排序分數），Redis 會自動根據 Score 進行升序或降序排列，極為適合排行榜。

```go
func zsetExample(rdb *redis.Client) {
	zsetKey := "leaderboard"

	// ====== ZADD (添加有分數的成員) ======
	err := rdb.ZAdd(ctx, zsetKey, redis.Z{
		Score:  1500.0,
		Member: "PlayerA",
	}, redis.Z{
		Score:  2300.0,
		Member: "PlayerB",
	}, redis.Z{
		Score:  1800.0,
		Member: "PlayerC",
	}).Err()
	if err != nil {
		log.Fatalf("ZAdd 失敗: %v", err)
	}

	// ====== ZRANGE (依照分數從小到大讀取) ======
	values, err := rdb.ZRangeWithScores(ctx, zsetKey, 0, -1).Result()
	if err != nil {
		log.Fatalf("ZRange 失敗: %v", err)
	}
	fmt.Println("排行榜 (從低到高):")
	for _, val := range values {
		fmt.Printf("  玩家: %v, 積分: %.1f\n", val.Member, val.Score)
	}

	// ====== ZREVRANGE (依照分數從大到小讀取 - 常用於排行榜前幾名) ======
	topPlayers, err := rdb.ZRevRangeWithScores(ctx, zsetKey, 0, 1).Result() // 拿前兩名
	if err != nil {
		log.Fatalf("ZRevRange 失敗: %v", err)
	}
	fmt.Println("前兩名高分玩家:")
	for _, val := range topPlayers {
		fmt.Printf("  玩家: %v, 積分: %.1f\n", val.Member, val.Score)
	}

	// ====== ZINCRBY (為特定成員增加分數) ======
	newScore, err := rdb.ZIncrBy(ctx, zsetKey, 500.0, "PlayerA").Result()
	if err != nil {
		log.Fatalf("ZIncrBy 失敗: %v", err)
	}
	fmt.Printf("PlayerA 贏得比賽，新積分: %.1f\n", newScore)

	// 清理整個 Key
	rdb.Del(ctx, zsetKey)
}
```
