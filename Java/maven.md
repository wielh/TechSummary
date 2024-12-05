
# Maven

## dependencies

+ 作用: 用於定義專案執行、編譯、測試等階段所需的外部函式庫。
+ 位置: 位於 project - dependencies 中。
+ 引入第三方函式庫，例如 Spring、Hibernate。定義依賴的作用域（scope）：
    + compile: 預設作用域，編譯和執行時都可用。
    + test: 僅在測試時可用。
    + runtime: 編譯時不可用，但執行時可用。
    + provided: 編譯時可用，但執行時間由容器提供。

<plugins>
+ 作用: 用於定義建置過程的插件，例如編譯、打包、部署等任務。
+ 位置: 位於 build - plugins中。
+ 定義專案建置過程中所需的插件。配置插件的行為，例如自訂編譯器版本、跳過測試等。
