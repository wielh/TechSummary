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

## 測試
1. JUnit（基礎測試）
    + 功能：測試單個方法是否回傳正確結果
    + 範例： 
        ```
        import org.junit.jupiter.api.Test;
        import static org.junit.jupiter.api.Assertions.*;

        public class MathTest {

            @Test
            void testAdd() {
                int sum = 2 + 3;
                assertEquals(5, sum);
            }

            @Test
            void testDivideByZero() {
                assertThrows(ArithmeticException.class, () -> {
                    int result = 10 / 0;
                });
            }
        }
        ```

2. Mockito（模擬依賴）
    + 功能：模擬 DAO、Service 等類別，不需要真實物件
    + 範例：
        ```
        import static org.mockito.Mockito.*;
        import static org.junit.jupiter.api.Assertions.*;

        import org.junit.jupiter.api.Test;
        import org.mockito.Mockito;

        public class UserServiceTest {

            @Test
            void testGetUserName() {
                UserRepository mockRepo = mock(UserRepository.class);
                when(mockRepo.getUserNameById(1L)).thenReturn("Alice");

                UserService userService = new UserService(mockRepo);
                String name = userService.getUserName(1L);

                assertEquals("Alice", name);
                verify(mockRepo).getUserNameById(1L); // 確認有被呼叫
            }
        }
        ```


3. Spring Test（整合測試）
    + 功能：啟動 Spring 上下文，可以測試 Controller、Service、Repository 的整體流程
    + 範例：
        ```
        @SpringBootTest
        @AutoConfigureMockMvc
        public class HelloControllerTest {

            @Autowired
            private MockMvc mockMvc;

            @Test
            void testHelloEndpoint() throws Exception {
                mockMvc.perform(get("/hello"))
                        .andExpect(status().isOk())
                        .andExpect(content().string("Hello, Spring Boot!"));
            }
        }
        ```
