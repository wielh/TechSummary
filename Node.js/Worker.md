# Worker 說明

## 說明

+ Worker 允許 Node.js 開啟多線程，適用於 CPU 密集的任務。

+ 主線程：Node.js 默認運行在單個主線程中。

+ Worker：Worker 是一個獨立的執行環境，擁有自己的事件循環。

+ Message Passing：主線程和 Worker 之間通過消息傳遞進行通信。

## 範例

+ main.js

```
const { Worker } = require('worker_threads');

const runService = (workerData) => {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./worker.js', { workerData });

    worker.on('message', resolve);
    worker.on('error', reject);
    worker.on('exit', (code) => {
      if (code !== 0)
        reject(new Error(`stopped with  ${code} exit code`));
    });
  })
}

const run = async () => {
  const result = await runService(999999);
  console.log(result);
}

run().catch(console.log);
```

+ worker.js

```
const { parentPort, workerData } = require('worker_threads');

const fibonacci = (n) => {
  var i;
  var fib = [];

  fib[0] = 0;
  fib[1] = 1;
  for (i = 2; i <= n; i++) {
    fib[i] = fib[i - 2] + fib[i - 1];
  }

  parentPort.postMessage(fib);
}

fibonacci(workerData);
```