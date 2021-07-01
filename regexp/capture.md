# 正则的捕获

## 实现正则捕获的方法

-   正则 RegExp.prototype 上的方法

    -   exec

        1. 结果是 null 或者一个数组，第一项：本次捕获的内容，其余项：对应小分组本次单独捕获的内容

            > index: 当前捕获内容在字符串中的起始索引。input：原始字符串

        2. 执行一次 exec 只能捕获到一个符合正则的结果，默认情况下执行多次结果也是第一个匹配到的，其余的捕获不到。 => 懒惰性: 默认只捕获第一个。

            > 懒惰性原因：正则属性 lastIndex: 当前正则下一次匹配的起始索引，默认 0 ,默认不会被修改。解决办法：g，参考下方示例。

    -   test

-   字符串 String.prototype 上支持正则表达式处理的方法

    -   replace
    -   match
    -   splite
    -   ...

```javascript
let str = "xxxxxxx2021qqqqqq2022yyyyyyyyy2021";
// 正则捕获前提是当前正则要和字符串匹配,不匹配结果为null
let reg = /\d+/;
reg.exec(str); // ["2021", index: 7, input: "xxxxxxx2021qqqqqq2022yyyyyyyyy2021", groups: undefined]

reg = /\d+/g; // 设置全局之后 会自动修改lastIndex
reg.exec(str); // 2021
reg.exec(str); // ["2022", index: 17, input: "xxxxxxx2021qqqqqq2022yyyyyyyyy2021", groups: undefined]
// 当捕获到最后位置(全部捕获完成之后再次捕获)结果为null，lastIndex重新赋值为0，从头开始。
```

## 正则捕获的贪婪性

-   默认情况，捕获按照匹配最长结果。
-   量词元字符后设置? 取消正则匹配的贪婪性，按照正则匹配最短的的结果匹配。

```javascript
let str = "xxxxx2021xxx2222xxx0000xxx3333";
let reg = /\d+/g;
str.match(reg); // [2021,2222,0000,3333]

reg = /\d+?/g; // 量词元字符后设置? 取消正则匹配的贪婪性，按照最短的
str.match(reg);
```

## 其他正则捕获

-   test

本意是匹配，但是也可以捕获。

```javascript
let str = "{0}年{1}月{2}日";
let reg = /\{(\d+)\}/g;
reg.test(str); // true
RegExp.$1; // '0'
reg.test(str); // true
RegExp.$1; // '1'
reg.test(str); // true
RegExp.$1; // '2'
reg.test(str); // false
RegExp.$1; // '2'

// RegExp.$1 - RegExp.$9 当前正则匹配后第一个到第九个分组信息。
```

-   [replace](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/replace)

字符串中实现替换的方法。一般都是伴随正则使用的。

str.replace(regexp|substr, newSubStr|function)

-   function 参数
    -   match:匹配的子串;
    -   p1,p2, ...: 则代表第 n 个括号(分组)匹配的字符串;
    -   offset: 匹配到的子字符串在原字符串中的偏移量。;
    -   string: 被匹配的原字符串。;
    -   NamedCaptureGroup: 命名捕获组匹配的对象

```javascript
let str = "xxxx2021xxxx3000";

str = str.replace("xxxx", "wwww"); // "wwww2021xxxx3000"
str = "xxxx2021xxxx3000";
str = str.replace(/xxxx/g, "wwww"); // "wwww2021wwww3000"
```

```javascript
let str = "xxxx2021xxxx3000";

str = str.replace("xxxx", "xxxxwwww"); // "xxxxwwww2021xxxx3000"
str = str.replace("xxxx", "xxxxwwww"); // "xxxxwwwwwwww2021xxxx3000"
str = "xxxx2021xxxx3000";
str = str.replace(/xxxx/g, "xxxxwwww"); // "xxxxwwww2021xxxxwwww3000"
```

-   处理时间字符串

```javascript
let date = "2021-07-01"; // 转换为 2021年07月01日
let reg = /^(\d{4})-(\d{1,2})-(\d{1,2})$/;
date.replace(reg, "$1年$2月$3日"); // "2021年07月01日"

// string.replace(regexp,function)
// 1. 首先拿regexp和string进行匹配不获，能匹配几次就会把传递的函数执行几次（匹配一次执行一次）
// 2. replace给方法传递参数(和exec捕获内容一致的信息: 大正则匹配的内容，小分组匹配的信息...)。
// 3. 该函数的返回值将替换掉第一个参数(匹配的子串)匹配到的结果。
date.replace(reg, (...args) => {
    // ["2021-07-01", "2021", "07", "01", 0, "2021-07-01"]
    let [b, $1, $2, $3] = args; // 2021-07-01 2021 07 01
    return `${$1}年${$2}月${$3}日`;
}); // "2021年07月01日"
```

-   单词首字母大写

```javascript
let str = "my name is secret344, are you ok!";
let reg = /\b([a-zA-Z])[a-zA-Z]*\b/g;
str.replace(reg, (...args) => {
    // (4)[("my", "m", 0, "my name is secret344, are you ok!")];
    // (4)[("name", "n", 3, "my name is secret344, are you ok!")];
    // (4)[("is", "i", 8, "my name is secret344, are you ok!")];
    // (4)[("are", "a", 22, "my name is secret344, are you ok!")];
    // (4)[("you", "y", 26, "my name is secret344, are you ok!")];
    // (4)[("ok", "o", 30, "my name is secret344, are you ok!")];
    let [content, $1] = args;
    $1 = $1.toUpperCase();
    content = content.substring(1);
    return $1 + content;
}); // "My Name Is secret344, Are You Ok!"
```
