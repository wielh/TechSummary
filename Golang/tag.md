# tag

## introduction
在 Go 語言中，tag 是結構體欄位中的一部分，主要用於存儲元數據，用於在執行時輔助反射操作。常見的用途包括 JSON 序列化、數據庫 ORM 映射、驗證、數據格式轉換等。

## Tag 的基本語法

```
type StructName struct {
    FieldName FieldType `tagKey1:"value1" tagKey2:"value2"`
}
```
+ FieldName：欄位名稱，需遵循大寫開頭才能被外部包訪問。
+ FieldType：欄位類型，例如 int、string 等。
+ tagKey1 和 tagKey2：Tag 的鍵（key），如 json、db。
+ value1 和 value2：Tag 的值（value）。

## 常見用途與範例

+ JSON 序列化與反序列化: 使用 json tag 指定結構體欄位與 JSON 中字段的映射關係：
```
package main

import (
    "encoding/json"
    "fmt"
)

type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email,omitempty"` // 如果值為空，則忽略該字段
}

func main() {
    user := User{ID: 1, Name: "Alice"}
    jsonData, _ := json.Marshal(user)
    fmt.Println(string(jsonData)) // {"id":1,"name":"Alice"}
}
omitempty：若欄位值為零值（如空字串或零），則序列化時忽略該字段。
```

+ GORM（ORM 映射）: 使用 gorm tag 指定資料庫表中的字段映射
```
package main

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
)

type User struct {
    ID    uint   `gorm:"primaryKey"`
    Name  string `gorm:"size:100;not null"`
    Email string `gorm:"unique"`
}

func main() {
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    db, _ := gorm.Open(mysql.Open(dsn), &gorm.Config{})

    db.AutoMigrate(&User{})
}
primaryKey：設置主鍵。
size：指定欄位長度。
unique：設置唯一約束。
```

+ Validation 驗證
使用 validate tag 指定欄位驗證規則：

```
package main

import (
    "fmt"
    "github.com/go-playground/validator/v10"
)

type User struct {
    Name  string `validate:"required"`
    Email string `validate:"required,email"`
    Age   int    `validate:"gte=0,lte=120"` // 年齡需介於 0 到 120 之間
}

func main() {
    validate := validator.New()
    user := User{Name: "", Email: "invalid", Age: 150}

    err := validate.Struct(user)
    if err != nil {
        fmt.Println("Validation errors:", err)
    }
}
```

+ 自定義 Tag

```
package main

import (
    "fmt"
    "reflect"
)

type User struct {
    Name  string `custom:"user_name"`
    Email string `custom:"user_email"`
}

func main() {
    user := User{Name: "Alice", Email: "alice@example.com"}
    t := reflect.TypeOf(user)

    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        fmt.Printf("Field: %s, Tag: %s\n", field.Name, field.Tag.Get("custom"))
    }
}
```

輸出：

```
Field: Name, Tag: user_name
Field: Email, Tag: user_email
```

+ Tag 的注意事項
    + 多個 Tag: 使用空格分隔多個 tag：
    + Tag 格式: Tag 必須以鍵值對形式表示，值需用雙引號括起。不符合格式的 tag 無法通過反射正確解析。
    + 反射解析:需使用標準庫 reflect 提取 tag。
    + 大小寫敏感:結構體欄位名需大寫開頭才能被反射訪問。