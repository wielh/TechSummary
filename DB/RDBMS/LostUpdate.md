# Lost Update

## 簡介
更新覆盖（Lost Update） 可能發生在併發情境下，通常是因為多個事務對同一筆資料進行修改時，未能正確處理並發更新，導致其中一個事務的更新被另一個事務覆蓋。

## go 範例 code

以下範例會輸出 800 or 900, 而不是預期的700。

```
func update1(id uint, reduce int, wg *sync.WaitGroup) {
	defer wg.Done()
	var user User
	db.First(&user, id)
	time.Sleep(1 * time.Second)
	user.Money = user.Money - reduce
	db.Save(&user)
}

func Update1Example() {
	var wg sync.WaitGroup
	wg.Add(2)
	go update1(1, 100, &wg)
	go update1(1, 200, &wg)
	wg.Wait()
}
```


## 可能原因

使用 select 取得試圖更新的的欄位，是取得他的快照值。這會導致在高併發情況下多個交易都針對同一份快照值更改，提交後會忽略其他交易的更改。

## 解決方案

+ 對於 sql 有支援的基本運算 (比如 col = col + 1)，可以直接單行更新，因為更新是使用當前讀。如以下範例:

```
UPDATE user SET money=money-100 WHERE id=1 AND money>=100
```

+ 使用隔離方案 SERIALIZABLE

+ 加上 column version，在程式面上實作樂觀鎖(與重試機制)

+ 改用 SELECT ... FOR UPDATE，以實現悲觀鎖