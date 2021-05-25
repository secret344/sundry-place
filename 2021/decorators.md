# Decorators 装饰器

> 以前就想对 ts 装饰器做个总结，但是长时间未使用也就搁置一边了，现在既然开始输出文章，那就总结一下吧。
>
> 本文其实是参考 typescript 的[装饰器原文](https://www.typescriptlang.org/docs/handbook/decorators.html)来总结的，想看原文的可以直接点击链接，移步官网。

## 产生原由

> 随着 TypeScript 和 ES6 中类的引入，现在有一些场景需要额外的功能来支持注释或修改类和类成员。 装饰器提供了一种为类声明和成员添加注释和元编程语法的方法。 装饰器是 JavaScript 的第二阶段建议，并作为 TypeScript 的一个实验性功能提供。

## 介绍

> 装饰器是一种特殊的声明，可以附加到类声明、方法、访问器、属性或参数上。 装饰器使用@expression 的形式，其中 expression 必须为一个函数，该函数将在运行时被调用，并带有关于被装饰的声明的信息。
>
> 以上是 typescript 官方的介绍，介绍的已经很好了，我们这里只研究使用。我们不介绍官方文章的[reflect-metadata](https://www.typescriptlang.org/docs/handbook/decorators.html#metadata)要想了解的可以自行去查阅。
>
> 最简单的示例,我们在使用的时候可以@sealed

```
function sealed(target) {
  // .......
}
```

## 装饰器构成

> 多个装饰器可以应用于同一个声明，例如在一行中 @f @g x。或者说在多行

```
@f
@g
x
```

> 当多个装饰器作用于同一个声明时，会执行以下步骤：

-   每个装饰器的表达式都会从上到下开始触发(评估)，
-   然后将结果作为函数从下到上调用。

> 我们来看一个官方的例子,我使用 Ⅰ Ⅱ Ⅲ Ⅳ 来标识打印顺序，这样更好理解。

```
function first() {
    console.log("first(): factory evaluated"); // Ⅰ
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("first(): called"); // Ⅳ
    };
}

function second() {
    console.log("second(): factory evaluated"); //  Ⅱ
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("second(): called"); //  Ⅲ
    };
}

class ExampleClass {
    @first()
    @second()
    method() {}
}

```

## 装饰器评估

> 对于应用在类内部各种声明上的装饰器，有一个明确的顺序。

1. 对于每个实例成员，首先是参数装饰器，然后是方法、Accessor(存取器、访问器)或属性装饰器。
2. 对于每个静态成员，先是参数装饰器，然后是方法、Accessor 或属性装饰器。
3. 参数装饰器被应用于构造函数。
4. 类的装饰者被应用于类

## 类装饰器（Class Decorators）

> 类装饰就是类声明之前被声明，类装饰器作用在类的构造函数上，用来观察，修改或者替换类定义。类装饰器不能在声明文件中使用，也不能在其他环境中使用（比如在 declare class 上）。
>
> 类装饰器的表达式在运行期间将作为函数被调用，被装饰的类的构造器是他唯一的参数。
> 如果返回一个值，将用提供的构造函数替换类的声明。
> 例子：

```
@sealed
class BugReport {
  type = "report";
  title: string;

  constructor(t: string) {
    this.title = t;
  }
}

function sealed(constructor: Function) {
  Object.seal(constructor); // 密封构造函数
  Object.seal(constructor.prototype);// 密封原型
}
```

> [Object.seal()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/seal),封闭一个对象，不可配置，如果之前属性可写，那么现在也可写。
>
> 上述就是封闭构造函数，原型这将不允许在运行时对类进行sub-classed。
TODO