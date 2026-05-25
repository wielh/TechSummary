# DDD（Domain-Driven Design，領域驅動設計）

## 簡介

DDD（Domain-Driven Design，領域驅動設計）是一種軟體開發的方法論與架構設計哲學，由 Eric Evans 提出，目的是讓複雜業務邏輯的程式碼與真實世界語言對齊，從而提高可維護性與協作效率。

## 一些概念與例子

+ **Entity (實體)**: Entity 為一具有屬性及行為的獨立事物，且此物件是必須被追蹤的實體。
  + 具有唯一的識別碼 (ID)，除識別碼外的其他狀態可變。
  + 兩個 Entity 不論其他狀態，ID 相同就是相同物件。
  + Entity 除了擁有 ID 及其屬性以外，還可以包含多個 Value Object 及 Entity。
  + **充血模型**是一種讓實體（Entity）自己承擔行為與狀態管理的模式，與**貧血模型**（Anemic Model）相對。以密碼為例，我們可以在它初始化的時候對其加入限制，也可以規定它的行為 (hash, match)。
  + **範例程式碼**
  
    ```java
        public class Password {
            private static final int MIN_LENGTH = 8;
            private static final int MAX_LENGTH = 100;
            private static final BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
            private final String password;

            public Password(String password) {
                if (password == null || password.isBlank()) {
                    throw new ParameterInvalidException("password not exist in parameter");
                }

                if (password.length() < MIN_LENGTH || password.length() > MAX_LENGTH) {
                    throw new ParameterInvalidException(String.format("Len of password should be between %d to %d", MIN_LENGTH, MAX_LENGTH));
                }

                this.password = password;
            }

            @Override
            public String toString(){
                return password;
            }

            public String hash() {
                return passwordEncoder.encode(this.password);
            }

            public boolean match(String hashedPassword) {
                return passwordEncoder.matches(this.password, hashedPassword);
            }
        }
        ```

+ **Value Object (值物件)**: 代表物件的特徵/屬性。當一個物件沒有概念上的標示，只關心它的屬性，該物件可建立成 Value Object。Value Object 可幫助 Domain Knowledge 的實現。(範例：將 Customer 中地址相關的屬性建立為 Address Value Object。)

+ **Aggregate (聚合)**: 聚合是業務和邏輯緊密關聯的 Entity 與 Value Object 的組合，幫助在複雜關聯的模型中確保物件更改的一致性，且更改物件均遵守業務規則。
  + 範例: 交易會牽涉到從用戶端扣除錢、扣除庫存與新增交易紀錄。
  + 透過 **Aggregate Root (聚合根)** 進行操作，外界無法得知除了 Aggregate Root 以外的實體，因此 Aggregate 邊界同時也是資料庫 **Transaction (交易)** 邊界。

## DDD 的好處

+ 更貼近真實業務語言（通用語言）。
+ 模型與程式碼一致、易維護。
+ 有清楚邊界（限界上下文），便於拆分微服務。
+ 複雜邏輯能封裝於聚合中（不散落各處）。
