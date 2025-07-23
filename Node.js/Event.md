# event loop 順序

在 Node.js 的事件迴圈 (Event Loop) 中，事件依據執行順序可以分為幾個階段，每個階段處理不同類型的事件

+ 執行順序: 同步 -> Microtask -> Macro Task
+ micro-task:
    + Promise.then()、Promise.catch() 和 Promise.finally() 會加入 Microtask 佇列。
    + queueMicrotask() 也可以放置自訂 Microtask
+ macro-task:
    + Timers 階段: 處理 setTimeout() 和 setInterval() 設定的計時器回調函數。
    + I/O callbacks 階段: 處理 上一輪的 I/O 事件回調 (callback)
    + Poll 階段: 處理新的 I/O 事件（如網路請求、文件操作等）。
    + Check 階段: 執行 setImmediate() 設定的回調。讓函數能夠在 I/O 發生後立即執行而不用等到下一輪 Timer 
    + Close Callbacks 階段: 處理關閉事件（如 socket.on('close')、server.close()）。