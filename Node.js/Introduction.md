# Node.js 特性

## 单线程事件驱动架构
Node.js 是单线程的，使用事件驱动的非阻塞 I/O 模型

## 非阻塞 I/O
Node.js 的非阻塞 I/O 特性允许在执行 I/O 操作时不阻塞线程，这使得它在处理 I/O 密集型任务时非常高效。

## NPM（Node Package Manager）
NPM 是 Node.js 的包管理工具，提供了丰富的第三方库和模块，可以轻松地添加到项目中。

## 流（Streams）
流是 Node.js 中处理 I/O 操作的一种抽象方式，特别适用于处理大量数据的情况下。
在 Node.js 中，流（Streams）是一种处理读写文件、网络通信或任何类型的端对端数据传输的高效方式。流提供了一种在不需要一次性将整个数据加载到内存中的情况下，逐块处理数据的机制。流主要分为四种类型：可读流（Readable）、可写流（Writable）、双工流（Duplex）和转换流

## 事件（Events）
Node.js 使用事件驱动模型，允许在特定事件发生时执行代码。以下是一些事件範例:
+ HTTP 服务器: 在 Node.js 中，我们可以创建 HTTP 服务器来响应客户端的请求。这些请求本质上就是事件，而服务器端则会注册相应的事件监听器来处理这些请求。
    example:
    ```
    const http = require('http');
    const server = http.createServer((req, res) => {
        res.writeHead(200, { 'Content-Type': 'text/plain' });
        res.end('Hello, Node.js!');
    });

    // 監聽 'request' 事件
    server.on('request', (req) => {
        console.log(`收到請求: ${req.url}`);
    });

    // 啟動伺服器
    server.listen(3000, () => console.log('伺服器運行於 http://localhost:3000'));
    ```

+ WebSocket: 在实时应用程序中，例如聊天应用、实时通知等，Node.js 的事件驱动模型和事件监听器非常适合处理客户端和服务器之间的双向通信，例如使用 WebSocket。
    example:
    ```
    const WebSocket = require('ws');
    const server = new WebSocket.Server({ port: 8080 });

    server.on('connection', (ws) => {
        console.log('有新的 WebSocket 連接');
        
        ws.send('歡迎加入！');

        ws.on('message', (message) => {
            console.log(`收到訊息: ${message}`);
        });

        ws.on('close', () => {
            console.log('連線已關閉');
        });
    });
    ```

+ IO操作: 例如讀取數據庫、檔案和調用第三方api。Node.js 可以通过事件和回调函数来处理数据库操作的结果。
    example
    ```
    const fs = require('fs');

    fs.readFile('file.txt', 'utf8', (err, data) => {
        if (err) throw err;
        console.log('文件內容:', data);
    });

    console.log('這行程式碼會先執行');
    ```

+ 自訂事件:
    ```
    const EventEmitter = require('events');
    class MyEmitter extends EventEmitter {}
    const myEmitter = new MyEmitter();
    // 監聽事件
    myEmitter.on('greet', (name) => {
        console.log(`Hello, ${name}!`);
    });

    // 觸發事件
    myEmitter.emit('greet', 'Alice');
    ```