# Jennifer - 強大的 Go 代碼生成工具

[Jennifer](https://github.com/dave/jennifer) 是一個無需範本 (Template-free) 的 Go 代碼生成庫。與使用 `text/template` 不同，Jennifer 透過流暢的 API (Fluent API) 讓開發者以撰寫 Go 的思維來生成代碼。

---

## 1. 為什麼選擇 Jennifer？

- **自動管理 Import**：這是 Jennifer 最強大的功能。你只需要指定 Package 路徑，它會自動處理別名衝突並在頂部生成 `import` 區塊。
- **避免語法錯誤**：使用範本時常會遇到少一個括號或縮排錯誤，Jennifer 的 API 設計確保了生成的代碼在結構上始終是合法的。
- **可讀性高**：代碼生成邏輯本身看起來就像是在寫 Go，非常直觀。
- **自動格式化**：生成時會自動處理縮排與換行。

---

## 2. 基本使用

### 建立一個簡單的 Hello World

```go
package main

import (
    "github.com/dave/jennifer/jen"
)

func main() {
    f := jen.NewFile("main")
    
    f.Func().Id("main").Params().Block(
        jen.Qual("fmt", "Println").Call(jen.Lit("Hello, Jennifer!")),
    )

    // 輸出到標準輸出或檔案
    // fmt.Printf("%#v", f)
    f.Save("hello_gen.go")
}
```

---

## 3. 核心 API 說明

### 標識符與字面量

- **`Id(name)`**：一般的標識符（變數名、函數名）。
- **`Qual(path, name)`**：具備 Package 路徑的標識符。Jennifer 會自動生成 import。
- **`Lit(v)`**：基本型別的字面量（String, Int, Bool 等）。

### 結構與流控制

- **`Func()`**：宣告函數。
- **`Params(...)`**：參數列表。
- **`Block(...)`**：代碼塊 `{ ... }`。
- **`If(...)`, `For(...)`, `Switch(...)`**：各種流程控制。

### 操作符與選擇器

- **`Op(op)`**：運算子（如 `:=`, `==`, `+`）。
- **`Dot(name)`**：成員存取（如 `obj.Field`）。
- **`Add(...)`**：將多個語句組合在一起。

---

## 4. 實戰範例對照 (Code vs. Output)

透過對比「生成邏輯 (Generator Code)」與「產出結果 (Generated Code)」，可以更直觀地理解 Jennifer 的運作方式。

### A. 產生常數結構 (Enum-like)

**生成邏輯：**

```go
f := jen.NewFile("models")
f.Var().Id("UserColumns").Op("=").Struct().Block(
    jen.Id("ID").Func().Params().String().Block(
        jen.Return(jen.Lit("user_id")),
    ),
)
```

**產出結果：**

```go
package models

var UserColumns = struct {
    ID func() string
}{
    ID: func() string {
        return "user_id"
    },
}
```

### B. 產生 Mock 實作

**生成邏輯：**

```go
f := jen.NewFile("mock")
f.Func().Params(
    jen.Id("s").Op("*").Id("MockService"),
).Id("GetData").Params(
    jen.Id("ctx").Qual("context", "Context"),
    jen.Id("req").Op("*").Id("Request"),
).Id("error").Block(
    jen.List(jen.Id("_"), jen.Id("err")).Op(":=").Id("run").Call(
        jen.Id("s.scripts"),
        jen.Id("req"),
    ),
    jen.Return(jen.Id("err")),
)
```

**產出結果：**

```go
package mock

import (
    "context"
)

func (s *MockService) GetData(ctx context.Context, req *Request) error {
    _, err := run(s.scripts, req)
    return err
}
```

### C. 產生 CRUD 邏輯 (配合泛型)

**生成邏輯：**

```go
f := jen.NewFile("service")
f.Func().Params(jen.Id("s").Id("service")).Id("Update").Params(
    jen.Id("ctx").Qual("context", "Context"),
    jen.Id("req").Id("UpdateReq"),
).Id("error").Block(
    jen.List(jen.Id("_"), jen.Id("err")).Op(":=").Qual("your-project/orm", "NewDB").
        Index(jen.Id("User")).
        Call(jen.Id("s.db")).
        Dot("Update").Call(jen.Id("ctx"), jen.Id("req")),
    jen.Return(jen.Id("err")),
)
```

**產出結果：**

```go
package service

import (
    "context"
    orm "your-project/orm"
)

func (s service) Update(ctx context.Context, req UpdateReq) error {
    _, err := orm.NewDB[User](s.db).Update(ctx, req)
    return err
}
```

---

## 5. 管理 Import 的技巧

Jennifer 的 `Qual` 方法是其核心競爭力：

```go
// 不需要手動寫 import "github.com/google/uuid"
f.Id("id").Op(":=").Qual("github.com/google/uuid", "New").Call()
```

如果多個 Package 有相同的名稱（例如 `your-project/errors` 與標準庫 `errors`），Jennifer 會自動為其中一個加上別名（如 `errors1`），確保代碼編譯成功。

---

## 6. 總結

Jennifer 是目前 Go 社群中最受歡迎的自定義生成器工具。它帶來的 **「自動 Import 管理」** 與 **「結構安全」** 是使用傳統 `text/template` 無法比擬的。在處理複雜的商業邏輯生成時，Jennifer 是確保產出代碼品質的首選工具。
