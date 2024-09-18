# Dependency injection 說明

## 簡介

根據依賴反轉原則，具體實現則應該依賴於抽象介面。我們必須了解到：並不是高階模組去依賴低階模組，而是高階模組提出它需要的功能，低階模組去實作出這些功能、達成高階模組的目標，這也比較接近我們開發程式時的思維。例如說我們需要「會員查詢」功能，才用「DB 連線方法」和「資料篩選方法」等具體方式去達成我們要「會員查詢」這個目標。「會員查詢」程式碼不應該因為「DB 連線方法」和「資料篩選方法」有所不同而更改。

## 情境

假设我们有一个 car class，其中包含各种对象，例如车轮、引擎等。
这里的 car class 负责创建所有依赖对象。现在，如果我们决定将来放弃 MRFWheels，而希望使用 Yokohama 车轮，该怎么办？

我们将需要使用新的 Yokohama 依赖关系来重新创建 car 对象。但是，当使用依赖注入（DI）时，我们可以在运行时更改车轮 wheels（因为可以在运行时而不是在编译时注入依赖项）。你可以将依赖注入视为代码中的中间人，它负责创建想要的 wheels 对象，并将其提供给car class。

它使 car class 不需要创建车轮 wheels、电池 battery 对象等。

##　依賴注入的種類

白話一點來說，「注入」也就是「丟進去」的意思。所以依賴注入就是指用各種方法把低階模組丟到高階模組裡。
主要常見的有三種作法：建構式注入、方法注入、屬性注入。也就是從建構式丟進去、從方法丟進去、從屬性丟進去。

## 優點
+ 測試性提升： 通過依賴注入，我們可以將虛擬的相依物件注入到類中，從而使單元測試更加輕鬆。這種方式下，我們可以專注於測試類的行為，而不需要實際創建相依物件。

+ 鬆散耦合： 依賴注入降低了類之間的耦合度，使不同模塊更加獨立。這使得程式碼更加靈活和易於修改。

+ 可維護性提升： 依賴注入使程式碼的組織更加清晰，類不再負責管理相依物件，使程式碼更易於理解和維護。

##　缺點
＋ 循環依賴：在使用依賴注入時，有時可能會出現循環依賴的情況，即不同的類彼此相互依賴，形成一個閉環。這種情況下，必須謹慎設計類之間的相依關係，以避免循環依賴導致程式碼難以理解和維護。