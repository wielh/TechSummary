# 如何使用 redis (golang為例子)

## list 

```
    // ====== CREATE ======
	// 在列表右側推入多個值
	err = rdb.RPush(ctx, listKey, "item1", "item2", "item3").Err()
	if err != nil {
		log.Fatalf("Failed to push values to list: %v", err)
	}
	fmt.Printf("List '%s' created with values: item1, item2, item3\n", listKey)

	// ====== READ ======
	// 獲取列表中所有元素
	values, err := rdb.LRange(ctx, listKey, 0, -1).Result()
	if err != nil {
		log.Fatalf("Failed to read list: %v", err)
	}
	fmt.Printf("Values in list '%s': %v\n", listKey, values)

	// ====== UPDATE ======
	// 更新列表中特定索引的值
	err = rdb.LSet(ctx, listKey, 1, "updated_item2").Err()
	if err != nil {
		log.Fatalf("Failed to update value in list: %v", err)
	}
	fmt.Printf("Updated index 1 in list '%s' to 'updated_item2'\n", listKey)

	// 再次獲取更新後的列表
	updatedValues, err := rdb.LRange(ctx, listKey, 0, -1).Result()
	if err != nil {
		log.Fatalf("Failed to read updated list: %v", err)
	}
	fmt.Printf("Updated values in list '%s': %v\n", listKey, updatedValues)

	// ====== DELETE ======
	// 從列表中移除特定值
	removedCount, err := rdb.LRem(ctx, listKey, 1, "item1").Result()
	if err != nil {
		log.Fatalf("Failed to remove value from list: %v", err)
	}
	fmt.Printf("Removed %d occurrence(s) of 'item1' from list '%s'\n", removedCount, listKey)

	// 獲取刪除後的列表
	finalValues, err := rdb.LRange(ctx, listKey, 0, -1).Result()
	if err != nil {
		log.Fatalf("Failed to read final list: %v", err)
	}
	fmt.Printf("Final values in list '%s': %v\n", listKey, finalValues)

	// ====== DELETE LIST ======
	// 刪除整個列表
	err = rdb.Del(ctx, listKey).Err()
	if err != nil {
		log.Fatalf("Failed to delete list: %v", err)
	}
	fmt.Printf("List '%s' deleted successfully.\n", listKey)
```

## hash 

```
	// --- CREATE: 新增或設置 Hash 值 ---
	err := rdb.HSet(ctx, hashKey, map[string]interface{}{
		"name":  "Alice",
		"age":   30,
		"email": "alice@example.com",
	}).Err()
	if err != nil {
		log.Fatalf("HSet failed: %v", err)
	}
	fmt.Println("Hash created successfully")

	// --- READ: 獲取 Hash 中的值 ---
	// 獲取單個字段的值
	name, err := rdb.HGet(ctx, hashKey, "name").Result()
	if err != nil {
		log.Fatalf("HGet failed: %v", err)
	}
	fmt.Printf("Name: %s\n", name)

	// 獲取所有字段和值
	hashData, err := rdb.HGetAll(ctx, hashKey).Result()
	if err != nil {
		log.Fatalf("HGetAll failed: %v", err)
	}
	fmt.Println("All Hash Data:", hashData)

	// --- UPDATE: 修改 Hash 中的字段值 ---
	err = rdb.HSet(ctx, hashKey, "age", 31).Err() // 更新字段
	if err != nil {
		log.Fatalf("HSet update failed: %v", err)
	}
	fmt.Println("Hash updated successfully")

	// 再次讀取更新後的值
	age, err := rdb.HGet(ctx, hashKey, "age").Result()
	if err != nil {
		log.Fatalf("HGet after update failed: %v", err)
	}
	fmt.Printf("Updated Age: %s\n", age)

	// --- DELETE: 刪除 Hash 的字段或整個鍵 ---
	// 刪除單個字段
	err = rdb.HDel(ctx, hashKey, "email").Err()
	if err != nil {
		log.Fatalf("HDel failed: %v", err)
	}
	fmt.Println("Field 'email' deleted successfully")

	// 刪除整個 Hash
	err = rdb.Del(ctx, hashKey).Err()
	if err != nil {
		log.Fatalf("Del failed: %v", err)
	}
	fmt.Println("Hash deleted successfully")
```

## zset

```
// ====== CREATE ======
	// 添加有序集合元素
	members := []*redis.Z{
		{Score: 1.0, Member: "item1"},
		{Score: 2.0, Member: "item2"},
		{Score: 3.0, Member: "item3"},
	}
	count, err := rdb.ZAdd(ctx, zsetKey, members...).Result()
	if err != nil {
		log.Fatalf("Failed to add elements to ZSet: %v", err)
	}
	fmt.Printf("Added %d elements to ZSet '%s'\n", count, zsetKey)

	// ====== READ ======
	// 獲取有序集合中的元素
	values, err := rdb.ZRangeWithScores(ctx, zsetKey, 0, -1).Result()
	if err != nil {
		log.Fatalf("Failed to read elements from ZSet: %v", err)
	}
	fmt.Printf("Elements in ZSet '%s':\n", zsetKey)
	for _, val := range values {
		fmt.Printf("  Member: %s, Score: %.1f\n", val.Member, val.Score)
	}

	// ====== UPDATE ======
	// 更新某個元素的分數
	newScore, err := rdb.ZIncrBy(ctx, zsetKey, 5.0, "item1").Result()
	if err != nil {
		log.Fatalf("Failed to update score in ZSet: %v", err)
	}
	fmt.Printf("Updated 'item1' score to %.1f\n", newScore)

	// 再次讀取更新後的元素
	updatedValues, err := rdb.ZRangeWithScores(ctx, zsetKey, 0, -1).Result()
	if err != nil {
		log.Fatalf("Failed to read updated elements from ZSet: %v", err)
	}
	fmt.Printf("Updated elements in ZSet '%s':\n", zsetKey)
	for _, val := range updatedValues {
		fmt.Printf("  Member: %s, Score: %.1f\n", val.Member, val.Score)
	}

	// ====== DELETE ======
	// 刪除某個元素
	removed, err := rdb.ZRem(ctx, zsetKey, "item2").Result()
	if err != nil {
		log.Fatalf("Failed to remove element from ZSet: %v", err)
	}
	fmt.Printf("Removed %d element(s) from ZSet '%s'\n", removed, zsetKey)

	// 刪除整個有序集合
	err = rdb.Del(ctx, zsetKey).Err()
	if err != nil {
		log.Fatalf("Failed to delete ZSet: %v", err)
	}
	fmt.Printf("ZSet '%s' deleted successfully.\n", zsetKey)
```

+ 創建 ZSet（CREATE）：使用 ZAdd 方法向有序集合添加多個元素。每個元素由 Score（排序分數）和 Member（成員值）組成。
+ 讀取 ZSet（READ）：使用 ZRangeWithScores 獲取有序集合的所有成員及其分數。0, -1 表示從頭到尾獲取整個集合。
+ 更新 ZSet（UPDATE）：使用 ZIncrBy 方法增加某個元素的分數。如果成員不存在，ZIncrBy 會自動創建該成員。
+ 刪除 ZSet 元素（DELETE - 部分）：使用 ZRem 方法移除指定成員。
+ 刪除整個 ZSet（DELETE - 全部）：使用 Del 方法刪除整個有序集合。

