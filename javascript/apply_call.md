# call 和 apply

## 概述

-   [call()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call) 方法使用一个指定的 this 值和单独给出的一个或多个参数来调用一个函数
-   [apply()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) 方法调用一个具有给定 this 值的函数，以及以一个数组（或类数组对象）的形式提供的参数。

## 参数

-   thisArg

    -   在 function 函数运行时使用的 this 值。请注意，this 可能不是该方法看到的实际值：如果这个函数处于非严格模式下，则指定为 null 或 undefined 时会自动替换为指向全局对象，原始值会被包装。

-   第二个参数
    -   arg1, arg2, ...
        > call :指定的参数列表。
    -   argsArray
        > apply :可选的。一个数组或者类数组对象，其中的数组元素将作为单独的参数传给 func 函数。如果该参数的值为 null 或 undefined，则表示不需要传入任何参数。从 ECMAScript 5 开始可以使用类数组对象.

## 实现 apply(call 同理，参数不同)

-   调用一个函数
    ```javascript
    function myapply(context) {
        if (typeof this !== "function") {
            throw new TypeError("Error");
        }
        this();
    }
    ```
-   使用一个指定的 this 值, 处于非严格模式下，则指定为 null 或 undefined 时会自动替换为指向全局对象
    ```javascript
    function myapply(context) {
        if (typeof this !== "function") {
            throw new TypeError("Error");
        }
        let self = context || window;
        self.fn = this;
        const result = self.fn();
        delete self.fn;
        return result;
    }
    ```
-   提供参数
    ```javascript
    function myapply(context, args) {
        if (typeof this !== "function") {
            throw new TypeError("Error");
        }
        let self = context || window;
        self.fn = this;
        const result = self.fn(...args);
        delete self.fn;
        return result;
    }
    ```

## 实现 call

```javascript
function mycall(context) {
    if (typeof this !== "function") {
        throw new TypeError("Error");
    }
    const [, args] = [...arguments];
    let self = context || window;
    self.fn = this;
    const result = self.fn(...args);
    delete self.fn;
    return result;
}
```
