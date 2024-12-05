# Callback hell

## Callback hell 範例

```
console.log("Start");

setTimeout(() => {
    console.log("Step 1");
    setTimeout(() => {
        console.log("Step 2");
        setTimeout(() => {
            console.log("Step 3");
            setTimeout(() => {
                console.log("Step 4");
                // 還可以繼續嵌套...
            }, 1000);
        }, 1000);
    }, 1000);
}, 1000);

console.log("End");

```

每個 setTimeout 依賴上一個的完成，導致層層嵌套，程式碼像「金字塔」一樣難以維護

## 解決方案一: 使用then

```
console.log("Start");

const step = (msg, time) => {
    return new Promise((resolve) => {
        setTimeout(() => {
            console.log(msg);
            resolve();
        }, time);
    });
};

step("Step 1", 1000)
    .then(() => step("Step 2", 1000))
    .then(() => step("Step 3", 1000))
    .then(() => step("Step 4", 1000))
    .catch((error) => console.error(error));

console.log("End");
```

## 解決方案二: 使用await/async
```
console.log("Start");

const step = (msg, time) => {
    return new Promise((resolve) => {
        setTimeout(() => {
            console.log(msg);
            resolve();
        }, time);
    });
};

const runSteps = async () => {
    await step("Step 1", 1000);
    await step("Step 2", 1000);
    await step("Step 3", 1000);
    await step("Step 4", 1000);
};

runSteps().catch((error) => console.error(error));

console.log("End");
```