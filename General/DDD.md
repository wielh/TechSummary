# DDD（Domain-Driven Design，領域驅動設計）

## 簡介
DDD（Domain-Driven Design，領域驅動設計）是一種軟體開發的方法論與架構設計哲學，由 Eric Evans 提出，目的是讓複雜業務邏輯的程式碼與真實世界語言對齊，從而提高可維護性與協作效率。

## 一些概念與例子
+ Entity: Entity為一具有屬性及行為的獨立事物，且此物件是必須被追蹤的實體。
    + 具有唯一的識別符(ID)，除識別符外的其他狀態可變
    + 兩個Entity不論其他狀態，ID相同就是相同物件
    + Entity 除了擁有ID及其屬性以外，還可以包含多個 Value Object 及 Entity。
    + 充血模型是一種讓實體（Entity）自己承擔行為與狀態管理的模式，與貧血模型（Anemic Model）相對。以密碼為例，我們可以在他初始化的時候對其加入限制。也可以規定他的行為(hash, match)
    + Example Code
        ```  
        public class Password {
            private static final int MIN_LENGTH = 8;
            private static final int MAX_LENGTH = 100;
            private static final BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
            private final String password;

            public Password(String password) {
                if (password == null || password.isBlank()) {
                    throw new ParameterInvaildExecption("password not exist in parameter");
                }

                if (password.length() < MIN_LENGTH || password.length() > MAX_LENGTH) {
                    throw new ParameterInvaildExecption(String.format("Len of password should be between %d to %d", MIN_LENGTH, MAX_LENGTH));
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

+ Value Object: 代表物件的特徵/屬性。當一個物件沒有概念上的標示，只關心它的屬性，該物件可建立成Value Object。Value Object可幫助Domain Knowledge的實現。(Example：將Customer中地址相關的屬性建立為Address Value Object。)

+ Aggregate: 聚合(Aggregate)是業務和邏輯緊密關聯的Entity與Value Object的組合，幫助在複雜關聯的模型中確保物件更改的一致性，且更改物件均遵守業務規則。
    + Example: 交易會牽涉到用戶從用戶扣除錢，扣除庫存與新增交易紀錄
    + 通過 Aggregate Root 進行操作，外界無法得知除了 Aggregate Root 以外的 entity，因此Aggregate邊界同時也是資料庫Transaction邊界。

## DDD 的好處
+ 更貼近真實業務語言（通用語言）
+ 模型與代碼一致、易維護
+ 有清楚邊界（限界上下文），便於拆分微服務
+ 複雜邏輯能封裝於聚合中（不散落各處）