# new

## 概述

> [new](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new) 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例。

## 语法

> new constructor[([arguments])]

## 参数

-   constructor 一个指定对象实例的类型的类或函数。
-   arguments 一个用于被 constructor 调用的参数列表。

## 描述

new 关键字会进行如下的操作：

-   创建一个空的简单 JavaScript 对象（即{}）；
-   链接该对象（设置该对象的 constructor）到另一个对象 ；
-   将步骤 1 新创建的对象作为 this 的上下文 ；
-   如果该函数没有返回对象，则返回 this。

## 实现

-   创建一个空的简单 JavaScript 对象（即{}）；

```javascript
function mynew() {
    let obj = Object.create({});
}
```

-   链接该对象（设置该对象的 constructor）到另一个对象 ；

```javascript
function mynew() {
    let obj = Object.create({});
    let context = [].shift.call(arguments);
    obj._proto_ = context.prototype;
}
```

-   将步骤 1 新创建的对象作为 this 的上下文 ；

```javascript
function mynew() {
    let obj = Object.create({});
    let context = [].shift.call(arguments);
    obj._proto_ = context.prototype;
    let result = context.apply(obj);
}
```

-   如果该函数没有返回对象，则返回 this。

```javascript
function mynew() {
    let obj = Object.create({});
    let context = [].shift.call(arguments);
    obj._proto_ = context.prototype;
    let result = context.apply(obj);
    return typeof result === "object" ? result : obj;
}
```

## tips:

-   prototype

    > Object.prototype 属性表示 Object 的原型对象

-   \_proto\_
    > **proto** 属性是一个访问器属性（一个 getter 函数和一个 setter 函数）, 暴露了通过它访问的对象的内部[[Prototype]] (一个对象或 null)。
    >
    > 更推荐使用 Object.getPrototypeOf/Reflect.getPrototypeOf 和 Object.setPrototypeOf/Reflect.setPrototypeOf
-   constructor
    > 返回创建实例对象的 Object 构造函数的引用。注意，此属性的值是对函数本身的引用，而不是一个包含函数名称的字符串。对原始类型来说，如 1，true 和"test"，该值只可读。
    >
    > 所有对象都会从它的原型上继承一个 constructor 属性
