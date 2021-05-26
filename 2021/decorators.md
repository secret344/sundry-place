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

```typescript
function sealed(target) {
    // .......
}
```

## 装饰器构成

> 多个装饰器可以应用于同一个声明，例如在一行中 @f @g x。或者说在多行

```typescript
@f
@g
x
```

> 当多个装饰器作用于同一个声明时，会执行以下步骤：

-   每个装饰器的表达式都会从上到下开始触发(评估)，
-   然后将结果作为函数从下到上调用。

> 我们来看一个官方的例子,我使用 Ⅰ Ⅱ Ⅲ Ⅳ 来标识打印顺序，这样更好理解。

```typescript
function first() {
    console.log("first(): factory evaluated"); // Ⅰ
    return function (
        target: any,
        propertyKey: string,
        descriptor: PropertyDescriptor
    ) {
        console.log("first(): called"); // Ⅳ
    };
}

function second() {
    console.log("second(): factory evaluated"); //  Ⅱ
    return function (
        target: any,
        propertyKey: string,
        descriptor: PropertyDescriptor
    ) {
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
4. 类的装饰器被应用于类

## 类装饰器（Class Decorators）

> 类装饰就是类声明之前被声明，类装饰器作用在类的构造函数上，用来观察，修改或者替换类定义。类装饰器不能在声明文件中使用，也不能在其他环境中使用（比如在 declare class 上）。
>
> 类装饰器的表达式在运行期间将作为函数被调用，被装饰的类的构造器是他唯一的参数。
> 如果返回一个值，将用提供的构造函数替换类的声明。

-   第一个例子

```typescript
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
    Object.seal(constructor.prototype); // 密封原型
}
```

> [Object.seal()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/seal),封闭一个对象，不可配置，如果之前属性可写，那么现在也可写。
>
> 上述就是封闭构造函数和它的原型，，这将不允许类在运行时被 sub-classed。

-   第二个例子

```typescript
function reportableClassDecorator<T extends { new (...args: any[]): {} }>(
    constructor: T
) {
    return class extends constructor {
        reportingURL = "http://www...";
    };
}

@reportableClassDecorator
class BugReport {
    type = "report";
    title: string;

    constructor(t: string) {
        this.title = t;
    }
}

const bug = new BugReport("Needs dark mode");
bug.reportingURL; // ts类型系统并不知道新属性，所以会报错.
```

> 上述例子我们覆盖了构造函数设置了新的默认值。

## 方法装饰器(Method Decorators)

> 方法装饰器就在方法声明之前被声明。 该装饰器被应用于方法的属性描述符，可以用来观察、修改或替换方法定义。 方法装饰器不能在声明文件中使用，不能在重载上使用，也不能在任何其他环境下使用（比如在 declare class 中）。
>
> 方法装饰器的表达式将在运行时作为一个函数被调用，有以下三个参数。

-   对于静态成员，是该类的构造函数，对于实例成员，是该类的原型。
-   成员的名称。
-   该成员的属性描述符。(如果 target 小于 ES5，那么是未定义的)。

> 如果方法装饰器返回一个值，它将被用作该方法的属性描述符。(如果 target 小于 ES5，返回值忽略)
>
> 例子

-   方法装饰器

```typescript
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }

    @enumerable(false)
    greet() {
        return "Hello, " + this.greeting;
    }
}

function enumerable(value: boolean) {
    return function (
        target: any,
        propertyKey: string,
        descriptor: PropertyDescriptor
    ) {
        descriptor.enumerable = value;
    };
}
```

> 上述就是一个方法装饰器的例子，我们也可以通过修改属性描述符的 value 来控制类方法的执行。

## 访问器装饰器（Accessor Decorators）

> 一个访问器装饰器就在访问器声明之前被声明。 访问器装饰器被应用到访问器的属性描述符上，可以用来观察、修改或替换访问器的定义。 访问器装饰器不能在声明文件中使用，也不能在任何其他环境下使用（比如在声明类中）。
>
> 访问器装饰器的表达式在运行时被调用，参数跟方法装饰器参数是一样的。如果 target 不小于 ES5，那么返回值将作为该成员的属性描述符。

-   TypeScript 不允许为一个成员的 get 和 set 访问器进行装饰。相反，该成员的所有装饰器必须应用于文件顺序中指定的第一个访问器。这是因为装饰器适用于一个属性描述符，它结合了 get 和 set 访问器，而不是每个声明。
-   例子

```typescript
class Point {
    private _x: number;
    private _y: number;
    constructor(x: number, y: number) {
        this._x = x;
        this._y = y;
    }

    @configurable(false)
    get x() {
        return this._x;
    }

    @configurable(false)
    get y() {
        return this._y;
    }
}
function configurable(value: boolean) {
    return function (
        target: any,
        propertyKey: string,
        descriptor: PropertyDescriptor
    ) {
        descriptor.configurable = value;
    };
}
```

## 属性装饰器(Property Decorators)

> 一个属性装饰器是在一个属性声明之前声明的。 一个属性装饰器不能在声明文件中使用，也不能在任何其他环境下使用（比如在 declare class 中）。

-   属性装饰器的表达式将在运行时作为一个函数被调用，有以下两个参数:
    -   对于静态成员，是该类的构造函数，对于实例成员，是该类的原型。
    -   成员的名称。
-   由于 TypeScript 中属性装饰器的初始化方式，属性描述符不作为属性装饰器的参数提供。这是因为目前没有机制在定义原型成员时描述一个实例属性，也没有办法观察或修改一个属性的初始化器。返回值也被忽略了。因此，一个属性装饰器只能用来观察一个类的特定名称的属性已经被声明。
-   我们可以使用这些信息来记录关于该属性的元数据，如下面的例子。
    -   [reflect-metadata](https://www.typescriptlang.org/docs/handbook/decorators.html#metadata)

```typescript
class Greeter {
    @format("Hello, %s")
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        let formatString = getFormat(this, "greeting");
        return formatString.replace("%s", this.greeting);
    }
}
```

```typescript
import "reflect-metadata"; // reflect-metadata库
const formatMetadataKey = Symbol("format");
function format(formatString: string) {
    // 添加一个元数据条目
    return Reflect.metadata(formatMetadataKey, formatString);
}
function getFormat(target: any, propertyKey: string) {
    // 读取元数据
    return Reflect.getMetadata(formatMetadataKey, target, propertyKey);
}
```

## 参数装饰器（Parameter Decorators）

> 参数装饰器就在参数声明之前被声明。 参数装饰器被应用于类构造器或方法声明的函数。 一个参数装饰器不能在声明文件、重载或任何其他环境中使用（比如在 declare class)。

-   参数装饰器的表达式将在运行时作为一个函数被调用，有以下三个参数。
    -   对于静态成员，是该类的构造函数，对于实例成员，是该类的原型。
    -   成员的名称。
    -   参数在函数的参数列表中的索引。
-   参数装饰器只能用来观察一个方法上已经声明的参数。
-   参数装饰器的返回值被忽略。
-   示例：

```typescript
class BugReport {
    type = "report";
    title: string;

    constructor(t: string) {
        this.title = t;
    }

    @validate
    print(@required verbose: boolean) {
        if (verbose) {
            return `type: ${this.type}\ntitle: ${this.title}`;
        } else {
            return this.title;
        }
    }
}
```

```typescript
import "reflect-metadata";
const requiredMetadataKey = Symbol("required");

function required(
    target: Object,
    propertyKey: string | symbol,
    parameterIndex: number
) {
    let existingRequiredParameters: number[] =
        Reflect.getOwnMetadata(requiredMetadataKey, target, propertyKey) || [];
    existingRequiredParameters.push(parameterIndex);
    Reflect.defineMetadata(
        requiredMetadataKey,
        existingRequiredParameters,
        target,
        propertyKey
    );
}

function validate(
    target: any,
    propertyName: string,
    descriptor: TypedPropertyDescriptor<Function>
) {
    let method = descriptor.value!;

    descriptor.value = function () {
        let requiredParameters: number[] = Reflect.getOwnMetadata(
            requiredMetadataKey,
            target,
            propertyName
        );
        if (requiredParameters) {
            for (let parameterIndex of requiredParameters) {
                if (
                    parameterIndex >= arguments.length ||
                    arguments[parameterIndex] === undefined
                ) {
                    throw new Error("Missing required argument.");
                }
            }
        }
        return method.apply(this, arguments);
    };
}
```

> @required 装饰器添加了一个元数据条目，将参数标记为必需。 然后，@validate 装饰器将现有的 greet 方法包装成一个函数，在调用原始方法之前验证参数。
