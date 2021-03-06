# 正则表达式

## 1.介绍

-   regular expression: RegExp
-   用来处理字符串的规则

    > 只能处理字符串
    > 他是一个规则：可以验证字符串是否符合某个规则(**test**),也可以吧字符串中符合规则的内容捕获到(**exec/match...**)

    ```javascript
    let str = "good good study, day day up!";
    // 学正则就是用来制定规则 (是否包含数字)
    let reg = /\d+/;
    reg.test(str); // => false

    str = "2019-08-12";
    reg.exec(str); // => ["2019", index:0, inputs:"原始字符串"]
    ```

## 2.编写正则表达式

-   创建方式有两种
    -   1、 字面量创建方式
        ```javascript
        //=>字面量创建方式 (两个斜杠之间包起来的,都是用来描述规则的元字符)
        let reg1 = /\d+/;
        ```
    -   2、构造函数模式创建
        ```javascript
        // 两个参数: 元字符字符串,修饰符字符串
        let reg2 = new RegExp("\\d+");
        ```
-   正则表达式存在变量

```javascript
// 两个斜杠之间包起来的都属于元字符
let type = "xxx";
let reg = /^@"+type+"@$/; // 错误方式  匹配 @"typeee"@

reg = new RegExp("^@" + type + "@$/");
```

-   正则表达式由两部分组成

    -   元字符
        > |       标签       | <center>含义<center>                                   |
        > | :--------------: | :----------------------------------------------------- |
        > | **_量词元字符_** | 设置出现的次数                                         |
        > |        \*        | 0 到多次                                               |
        > |        +         | 1 到多次                                               |
        > |        ？        | 0 次或 1 次                                            |
        > |       {n}        | 出现 n 次                                              |
        > |       {n,}       | 出现 n 到多次                                          |
        > |      {n,m}       | 出现 n 到 m 次 包含 n,m                                |
        > | **_特殊元字符_** | 单个或者组合在一起代表特殊的含义                       |
        > |        \         | 转义字符 普通 -> 特殊 -> 普通                          |
        > |        .         | 除\n **(换行符)** 以外的任意字符                       |
        > |        ^         | 以哪一个元字符作为开始                                 |
        > |        $         | 以哪一个元字符作为结束                                 |
        > |        \n        | 换行符                                                 |
        > |        \d        | 0-9 之间的一个数字                                     |
        > |        \D        | 非 0-9 之间的一个数字 **（大写和小写的意思是相反的）** |
        > |        \w        | 数字、字母、下划线中的任意一个字符                     |
        > |        \s        | 一个空白字符 **（包含空格、制表符、换页符等）**        |
        > |        \t        | 一个制表符 **(一个 TAB 键：四个空格)**                 |
        > |        \b        | 匹配一个单词的边界                                     |
        > |       x\|y       | x 或者 y 中的一个字符                                  |
        > |      [xyz]       | x 或者 y 或者 z 中的一个字符                           |
        > |      [^xy]       | 除了 x/y 以外的任意字符                                |
        > |      [a-z]       | 指定 a-z 这个范围中的任意字符 eg: [0-9a-zA-Z_] === \w  |
        > |      [^a-z]      | 上一个的取反"非"                                       |
        > |        ()        | 正则中的分组符号                                       |
        > |       (?:)       | 只匹配不捕获                                           |
        > |       (?=)       | 正向预查                                               |
        > |       (?!)       | 负向预查                                               |
        > | **_普通元字符_** | 代表本身含义                                           |
        > |       /a/        | 此正则匹配的就是 a                                     |
    -   修饰符 img\*/

        > | 标签 | <center>含义<center>          |
        > | :--: | :---------------------------- |
        > |  i   | ignoreCase 忽略单词大小写匹配 |
        > |  m   | multiline 可以进行多行匹配    |
        > |  g   | global 全局匹配               |
        >
        > eg:
        >
        > ```javascript
        > /A/.test("lalala"); // false
        > /A/i.test("lalala"); //true
        > ```
