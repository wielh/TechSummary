# Java 中實作 Async 的方式

## 說明

非同步（async）程式設計是指任務在背景執行，不會阻塞主執行緒，結果在未來某一時刻可取得。 這種設計對於提升效能與響應速度非常重要，特別是在處理 IO 密集任務（如 HTTP、資料庫） 時。

## CompletableFuture（Java 8+）

+ example1

    ```
    CompletableFuture.runAsync(() -> {
        System.out.println("執行非同步任務");
    });
    ```

+ example2

    ```
    CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
        return 10 * 2;
    });
    future.thenAccept(result -> System.out.println("結果: " + result));
    ```

## @Async（Spring Boot 提供）

+ example1

    ```
    @Service
    public class MyService {

        @Async
        public void sendEmail(String address) {
            // 模擬寄信
            Thread.sleep(2000);
            System.out.println("已寄信給：" + address);
        }
    }
    ```

+ 注意事項
   + @Async 必須在 Spring 管理的 Bean 中使用	
   + @Async 不能用在 同一類別中自呼叫的方法	
   + 可以自訂 TaskExecutor 控制執行緒池數量