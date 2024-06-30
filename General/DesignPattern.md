# 設計模式原則

## 原則

1. 單一職責原則（Single Responsibility Principle，SRP）：
    一個類別應該只有一個造成變化的原因，即一個類別應該只有一個職責。這意味著一個類別應該只負責一組相關的功能，如果一個類別承擔了太多的職責，就會變得難以理解、修改和測試。

2. 開閉原則（Open/Closed Principle，OCP）：
    軟體實體（類別、模組、函數等）應該對擴充開放，對修改關閉。這意味著在不修改現有程式碼的情況下，應該能夠透過擴充功能來添加新的功能或改變現有功能的行為。

3. 里氏替換原則（Liskov Substitution Principle，LSP）：
    所有引用基底類別的地方必須能夠透明地使用其子類別的對象，即子類別可以替換父類別並且不會影響程式的正確性。這意味著子類別應該符合父類所製定的契約或規範。

4. 介面隔離原則（Interface Segregation Principle，ISP）：
    客戶端不應該被迫依賴它不使用的接口，即應該將大接口拆分成更小的、更具體的接口，以便客戶端只需要知道它們感興趣的接口。

5. 依賴倒置原則（Dependency Inversion Principle，DIP）：
    高層模組不應該依賴低層模組，二者都應該依賴抽象；抽像不應該依賴具體實現，具體實現應該依賴抽象。這意味著應該透過介面和抽象來解耦模組之間的依賴關係。

6. 合成/聚合復用原則（Composite/Aggregate Reuse Principle，CARP）：
    盡量使用合成/聚合，盡量不要使用繼承。這意味著應該透過組合（合成）和聚合來實現程式碼的複用，而不是透過繼承來實現。

7. 最少知識原則（Law of Demeter，LoD）：
    一個物件應該對其他物件有盡可能少的了解，不應該直接呼叫其他物件的方法，而是應該透過自己的成員方法來存取其他物件。

## 模式

+ 創建型模式（Creational）

    + 單例模式

    + 工廠模式

    + 原型模式

    + 建造者模式（Builder）

+ 結構型模式（Structural）

    + 代理模式（Proxy）	

    + 門面模式（Facade）	

    + 裝飾器模式（Decorator）

    + 享元模式（Flyweight）

    + 組合模式（Composite）	

    + 適配器模式（Adapter）

    + 橋接模式（Bridge）	

+ 行為型模式（Behavioral）

    + 委派模式（Delegate）	

    + 模板模式（Template）	

    + 策略模式（Strategy）	

    + 責任鏈模式（Chain of Responsibility）	

    + 迭代器模式（Iterator）

    + 命令模式（Command）

    + 狀態模式（State）	

    + 備忘錄模式（Memento）	

    + 中介者模式（Mediator）

    + 解釋器模式（Interpreter）	

    + 觀察者模式（Observer）	

    + 訪問者模式（Visitor）	