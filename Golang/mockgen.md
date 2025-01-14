# mockgen 

## 特性與功能

功能: 根據文件設計的介面生成對應的 mock 實例

基於反射生成：反射可以用來在運行時動態生成 mock 類。這種方式適合接口定義和實現分離的情況。

## 為何要這樣做

```
type function interface {
    f(x int)int
    g(x int)int
    h(x int)int
}

func calculate(x int) {
    for i:=0; i<n ; i++ {
        x = f(x)
    }

    if x > 0 {
        return g(x)
    }
    return h(x)
}
```

以上面的程式作為範例，對程式做 unit test，除了要測試 f,g,h 外，也要測試calculate。但假設f,g,h是依賴外部資源(
比如DB，第三方API)，則我們需要重新編寫一個新實例，用舉例的方式 (F1= f(x1), ... Fn=f(xn)) 模擬f,g,h的行為。
以用來測試calculate。

## 指令

+ 安裝：

```
go install github.com/golang/mock/mockgen@latest

```

+ 生成 mock 文件: 

```
mockgen -source={path} -destination={path} -package={name}

```

+ 範例

```
func TestMyService(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()

    mockService := mocks.NewMockMyService(ctrl)

    // 設定期望值
    mockService.EXPECT().f(1).Return(1)
    mockService.EXPECT().f(2).Return(2)
    mockService.EXPECT().f(3).Return(3)

    // 設定期望值
    mockService.EXPECT().g(1).Return(0)
    mockService.EXPECT().g(2).Return(1)
    mockService.EXPECT().g(3).Return(-1)

    // 設定期望值
    mockService.EXPECT().h(0).Return(0)
    mockService.EXPECT().h(2).Return(0.02)
    mockService.EXPECT().h(4).Return(5)

    // 調用測試方法
    result := calculate(1)
    if result != 0 {
        t.Errorf("unexpected result: %v", result)
    }
}
```