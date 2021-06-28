# 利用 Promise.race 实现一个可控异步并发数量请求函数

利用 Promise.race 实现一个可控异步并发数量请求函数，同时进行 N 个请求，直到所有请求结束。

## Promise.race

Promise.race(iterable) 方法返回一个 promise，一旦迭代器中的某个 promise 解决或拒绝，返回的 promise 就会解决或拒绝。

## 测试用例

-   模拟请求
    -   timeout： 模拟请求延迟
    -   data： 请求结果

```javascript
// 模拟生成请求
function promiseCreator({ timeout, data }) {
    return () => {
        return new Promise((res, rej) => {
            setTimeout(() => {
                res(data);
            }, timeout);
        });
    };
}
```

## 实现

-   根据要求，我们需要一个接受 2 个参数的函数

    -   limit： 控制并发数量
    -   fetchArr：请求作为数组传入

-   开始实现，创建函数

    ```javascript
    function fetchController(limit, fetchArr) {}
    ```

-   我们需要根 limit 进行并发控制，所以需要先提取请求交给 Promise.race 去执行

    ```javascript
    function fetchController(limit, fetchArr) {
        let baseHttp = fetchArr.splice(0, limit);
        let promise = Promise.race(baseHttp.map((item) => item()));
    }
    ```

-   此时开始执行并发请求，我们需要等待某个请求结束的同时进行下一个请求，将正在进行的请求与新开始的请求生成新的 Promise.race 执行，等待下一个请求函数完成。

    ```javascript
    function fetchController(limit, fetchArr) {
        let baseHttp = fetchArr.splice(0, limit);
        let promise = Promise.race(baseHttp.map((item) => item()));
        for (let i = 0; i < fetchArr.length; i++) {
            const ele = fetchArr[i];
            // 模拟实现
            // 假设之前race某个请求结束，这个请求在baseHttp的索引为 xxx
            // 我们只需要替换掉这个已经完成的请求，然后重新生成一个race就可以继续等待随后的请求结束，重复这个步骤
            // 直到请求全部完成
            baseHttp[xxx] = ele;
            promise = Promise.race(baseHttp);
        }
    }
    ```

-   从上面，我们可以发现，我们需要记录每一个请求结束时对应 baseHttp 数组内的索引，以此来重新生成新的 Promise.race，代码如下：

    ```javascript
    function fetchController(limit, fetchArr) {
        let baseHttp = fetchArr.splice(0, limit);
        let promise = Promise.race(
            baseHttp.map((item, index) => {
                return item().then((res) => {
                    // 利用链式调用，返回索引
                    return index;
                });
            })
        );
        for (let i = 0; i < fetchArr.length; i++) {
            const ele = fetchArr[i];
            promise = promise.then((res) => {
                // 根据promise的链式调用，此时res值就为执行完成的请求在baseHttp下的索引。
                let index = res;
                baseHttp[index] = ele.then((res) => {
                    return index;
                });
                // 此时我们返回新的race等待下个请求结束，继续执行该promise.then
                // 链式模拟如下 promise.race -> promise.then -> promise.race ...
                return Promise.race(baseHttp);
            });
        }
    }
    ```

## 整理代码

    ```javascript
        function fetchController(limit, fetchArr) {
            // promise化所有请求，根据实际自行增删，这里只是想到就写了
            // 保留了原本请求结束的结果，可以利用这个实现Promise.all 类似效果（除非原本请求没有接受请求结果，或者原本请求接收请求结果之后return 了这个结果，原因参见promise的链式调用）
            let createPromise = (ele, index) => {
                return Promise.resolve(ele()).then(
                    (res) => {
                        return { res, index };
                    },
                    (rej) => {
                        return { rej, index };
                    }
                );
            };

            let baseHttp = fetchArr
                .splice(0, limit)
                .map((item, index) => createPromise(item, index));

            let promise = Promise.race(baseHttp);
            for (let i = 0; i < fetchArr.length; i++) {
                const ele = fetchArr[i];
                let finally = (res) => {
                    let index = res.index;
                    baseHttp[index] = createPromise(ele, index);
                    return Promise.race(baseHttp);
                };
                promise = promise.then(finally, finally);
            }
        }
    ```

## 调用

```javascript
let base = [
    {
        timeout: 1000,
        data: 1,
    },
    {
        timeout: 500,
        data: 2,
    },
    {
        timeout: 3000,
        data: 3,
    },
    {
        timeout: 1200,
        data: 4,
    },
    {
        timeout: 3200,
        data: 5,
    },
    {
        timeout: 2200,
        data: 6,
    },
];

fetchController(
    2,
    base.map((item) => promiseCreator(item))
);
// 结果 2 1 4 3 5 6
```

## Promise 的链式调用

// TODO
