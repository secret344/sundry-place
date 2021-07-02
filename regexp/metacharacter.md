# 部分元字符详细解析

## 1.^ $

> ```javascript
> let reg = /^\d/;
> reg.test("asd"); // false
> reg.test("2019asd"); // true
>
> let reg = /\d$/;
> reg.test("asd"); // false
> reg.test("2019asd123"); // true
>
> // ^ $ 都不加 包含符合规则就行
> let reg1 = /\d+/; // 包含数字就行
> // 只能是和规则一致的内容
> let reg2 = /^\d+$/; //只能是一到多个数字开头 一到多个数字结尾
>
> // eg: 手机号 11位 第一个数字是1即可
> let reg = /^1\d{10}$/;
> ```

## 2.转义字符 \\

> ```javascript
> let reg = /^2.3$/; // . 除\n以外的任意字符 不是小数点
> reg.test("2.3"); // true
> reg.test("2@3"); // true
> reg.test("23"); // false
> let reg = /^2\.3$/; // 匹配2.3 基于转义字符\,让其只能代表小数点
>
> let str = "\\d"; // 不能\d
> let reg = /^\\d$/; //特殊符号转为普通的
> reg.test(str); // true
> ```

## 3. x | y

> ```javascript
> let reg = /^18|29$/;
> reg.test("18"); // true
> reg.test("189"); // true
> reg.test("1829"); // true
> reg.test("82"); // false
>
> // --- 直接 x | y 会存在 优先级问题，一般我们写的时候都伴随着小括号进行分组
> // 因为小括号能改变处理的优先级 => 小括号 : 分组
> reg = /^(18|29)$/; //只能是18或者29中的一个
> reg.test("18"); // true
> reg.test("189"); // false
> ```

## 4. []

-   中括号出现的字符 一般都代表本身含义

    > ```javascript
    > let reg = /^[@+]+$/;
    > reg.test("@@"); // true
    > reg.test("@+"); // true
    >
    > reg = /^[\d]$/; // \d 代表 还是0-9
    > ```

-   中括号中不存在多位数
    > ```javascript
    > let reg = /^[18]$/;
    > reg.test("1"); // true
    > reg.test("18"); // false
    >
    > reg = /^[10-29]$/; // 1或者 0-2 或者9
    > reg.test("0"); // true
    > reg.test("1"); // true
    > reg.test("2"); // true
    > reg.test("9"); // true
    > reg.test("10"); // false
    > ```

## 常用的正则表达式

-   验证是否为有效数字
    -   可能出现 + - 号,也可能不出现
    -   一位 0-9 都可以，多位首位不能为 0
    -   小数部分可能有可能没有，一旦有后面必须有小数点加数字

```javascript
let reg = /^[+-]?(\d|([1-9]\d+))(\.\d+)?$/;
```

-   验证密码
    -   数字字母下划线
    -   6-16 位

```javascript
let reg = /^\w{6,16}$/;
```

-   验证真实姓名
    -   汉字 /^[\u4E00-\u9FA5]$/
    -   名字长度暂定 2-10
    -   译名存在 ·xx

```javascript
let reg = /^[\u4E00-\u9FA5]{2,10}(·[\u4E00-\u9FA5]{2,10}){0,2}$/;
```

-   邮箱

    ```javascript
    let reg =
        /^\w+((-\w+)|(\.\w+))*@[A-Za-z0-9]+((\.|-)[A-Za-z0-9]+)*\.[A-Za-z0-9]+$/;
    ```

    -   开头 数字字母下划线 -xxx 或者.xxx

        eg: zhao-xx-xx@; zhao.xx.xx@; zhao-xx.xx@

    -   @
    -   数字字母 多个.xxx 或者-xxx

        eg: @xxx.com.cn;

        -   企业域名

            eg: xx@xx-xx-xx.com

    -   结尾 .xxx
        eg: .com; .cn; .org; .edu; .vip; .net; .xyz

-   身份证号码
    -   18 位
    -   最后一位可能会有 X
    -   前六位： 省市县
    -   中 8 位： 生产日期
    -   后 4 位：
        -   倒数第 2 性别 奇数男 偶数女
        -   最后一位 X 数字
        -   其余算法计算

```javascript
let reg = /^\d{17}(\d|X)$/;
// ()分组捕获
let reg = /^(\d{6})(\d{4})(\d{2})(\d{2})\d{2}(\d)(\d|X)$/;
reg.exec("632138199603036543"); // 瞎编的
```

## ?

-   本身代表出现 0 到 1 次
-   左边是量词元字符，取消捕获时贪婪性
-   (?:) 只匹配 不捕获
-   (?=) 正向预查
-   (?!) 负向预查
