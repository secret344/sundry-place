# bind

## 概述([mdn](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind))

> bind() 方法创建一个新的函数，在 bind() 被调用时，这个新函数的 this 被指定为 bind() 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。

## 语法([mdn](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind))

> function.bind(thisArg[, arg1[, arg2[, ...]]])

## 参数([mdn](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind))

-   thisArg
    > 调用绑定函数时作为 this 参数传递给目标函数的值。 如果使用 new 运算符构造绑定函数，则忽略该值。当使用 bind 在 setTimeout 中创建一个函数（作为回调提供）时，作为 thisArg 传递的任何原始值都将转换为 object。如果 bind 函数的参数列表为空，或者 thisArg 是 null 或 undefined，执行作用域的 this 将被视为新函数的 thisArg。
-   arg1, arg2, ...
    > 当目标函数被调用时，被预置入绑定函数的参数列表中的参数。

---

## 以上是 mdn 对 bind 的解释，我认为了解概念应该去 mdn，我们仿照实现一个 bind 函数.

-   bind()函数创建一个新的函数。

```javascript
function bind() {
    return function () {};
}
```

-   在 bind() 被调用时，这个新函数的 this 被指定为 bind() 的第一个参数，

```javascript
function bind(context) {
    let self = this;
    return function () {
        return self.apply(context);
    };
}
```

-   其余参数将作为新函数的参数，供调用时使用。

```javascript
function bind(context, ...args) {
    let self = this;
    return function () {
        return self.apply(context, [...arguments, ...args]);
    };
}
```

-   调用绑定函数时作为 this 参数传递给目标函数的值。 如果使用 new 运算符构造绑定函数，则忽略该值。

```javascript
function bind(context, ...args) {
    let self = this;
    let boundFun = function () {
        if (this instanceof boundFun) {
            return self.apply(this, [...arguments, ...args]);
        }
        return self.apply(context, [...arguments, ...args]);
    };
    boundFun.prototype = this.prototype;
    return boundFun;
}
```

-   优化

```javascript
let ERROR_MESSAGE = "Function.prototype.bind called on incompatible ";
let funcType = "[object Function]";
function bind(context, ...args) {
    let self = this;
    if (typeof self !== "function" || toStr.call(self) !== funcType) {
        // bind 作用于函数
        throw new TypeError(ERROR_MESSAGE + self);
    }
    let Empty = function Empty() {};
    let boundFun = function () {
        // boundFun的prototype在后面被我们赋值为Empty的实例
        if (this instanceof Empty) {
            return self.apply(this, [...arguments, ...args]);
        }
        return self.apply(context, [...arguments, ...args]);
    };
    // 为了修改boundFun.prototype不会影响原绑定函数
    Empty.prototype = self.prototype;
    boundFun.prototype = new Empty();
    Empty.prototype = null;

    return boundFun;
}
```

<strong style="color:red;">tips:代码未实际测试</strong>
