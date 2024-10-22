# special return value

## undefined
undefined 是一個表示「未定義」狀態的特殊值。它有多種用途和特性

+ 變量聲明但未賦值
+ 函數沒有返回值：當函數沒有顯式返回值時，它默認返回 undefined。
+ 對象屬性不存在：當訪問對象中不存在的屬性時，結果是 undefined。
+ 數組元素未初始化：當數組元素未初始化時，其值是 undefined。
+ 函數參數未傳遞：當函數被調用時，未傳遞的參數值是 undefined。

## any

+ any 類型表示任何類型，可以用來表示動態內容或不確定的類型。
+ 使用 any 可以完全跳過類型檢查，這在需要快速轉換或集成舊代碼時非常有用。
+ 任何類型的值都可以賦給 any 類型的變量，反之亦然。
+ 使用 any 類型的變量可以調用任何屬性或方法，而不會引發編譯錯誤。
+ 過度使用 any 可能會導致類型安全性降低，失去 TypeScript 帶來的靜態類型檢查優勢。

## unknown

+ unknown 類型表示未知的類型，類似於 any，但更加安全。
+ unknown 可以用來表示不確定的類型，通常用於類型安全的場景下，例如接收動態輸入或返回不確定類型的值。
+ 任何類型的值都可以賦給 unknown 類型的變量。
+ 在對 unknown 類型的變量進行操作之前，需要進行類型檢查或斷言，這可以提供更好的類型安全性。
+ 與 any 不同，不能直接對 unknown 類型的變量調用屬性或方法。

```
let value: unknown;
value = "hello";
value = 42;

if (typeof value === "string") {
    console.log(value.toUpperCase()); // 這是安全的
}

let anotherValue: unknown = value as string;
console.log((anotherValue as string).toUpperCase()); // 需要進行類型斷言
```

## never

+ never 類型表示永遠不會發生的值。用於那些不應該有返回值的函數，或者會拋出錯誤或無限循環的函數。
+ 常用於確保函數不會正常返回，用於類型保證，例如處理不可能發生的情況。
+ never 是所有類型的子類型，可以賦值給任何類型，但沒有任何類型是 never 的子類型，因此沒有值可以賦值給 never。
