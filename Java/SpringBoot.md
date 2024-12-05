# SpringBoot

## 核心概念

+  Bean: Bean 是一個普通的 Java 對象，但它由 Spring 容器管理。這意味著你可以將應用程式中的關鍵組件定義為 Bean，並依賴容器來管理它們的依賴性與生命周期。
+  @Component 是一種標記，告訴 Spring 容器這是一個需要管理的 Bean。其他派生註解包括：
    + @Service: 用於服務層。
    + @Repository: 用於數據訪問層。
    + @Controller / @RestController: 用於控制層。
+ 自動掃描: Spring Boot 通過 @SpringBootApplication 或 @ComponentScan 自動掃描指定包中的 @Component 類，並將其註冊為 Bean。
+ AutoWired: 對變數使用@AutoWired， spring 會從 Bean中挑選適合的類別注入。

## 架構

+ src/main/java: 存放应用程序的 Java 代码。
+ src/main/resources: 存放配置文件、静态资源、模板文件等。
+ application.properties 或 application.yml: 配置文件。
+ pom.xml 或 build.gradle: 构建文件，用于管理依赖。
