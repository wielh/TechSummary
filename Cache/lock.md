# 使用 redis 設計方案

## example1: 確認同一時間只能有一個程序訪問資源(單redis)

+ 設置鎖的指令(redis command)
    ```
    SET {key} {value} NX EX {second}
    ```

+ 刪除鎖的指令(Lua)
    ```
    if redis.call("GET", KEYS[1]) == ARGV[1] then
        return redis.call("DEL", KEYS[1])
    else
        return 0
    end
    ```

+ 程序們都試著執行以上命令，只有一個能成功取得鎖並執行程序。等待程序執行完畢或超時後釋放資源。
value 的值由獲得鎖的線程隨機機定，可以起到"只有獲取鎖的線程才能成功刪除鎖"的作用。


## example2: 基於 Lua 腳本的可重入鎖

## example3: 基於 ZSet（有序集合）的隊列鎖

## example4: 通知鎖，不用藉由輪詢獲得鎖
