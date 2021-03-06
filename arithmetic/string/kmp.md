# KMP 算法

> Knuth-Morris-Pratt 字符串查找算法（简称为 KMP 算法）可在一个字符串 S 内查找一个词 W 的出现位置。一个词在不匹配时本身就包含足够的信息来确定下一个匹配可能的开始位置，此算法利用这一特性以避免重新检查先前匹配的字符。

## 查找示例

> 首先我们以 kmp 算法的查找过程来了解下 kmp.
>
> 我们以 W="ABCDABD"，S="ABC ABCDAB ABCDABCDABDE"为例来讲述查找过程

```
    index: 01234567890123456789012
        S: ABC ABCDAB ABCDABCDABDE
        W: ABCDABD
        i: 0123456
```

开始查找，以上可知当索引为 3（D）的时候，第一次匹配失败，此时按照我们普通的查找方式，后移一位然后继续执行匹配过程，我们很明显的就可以知道普通的查找时间复杂度最坏为 n \* m,(最坏示例 S: AAAAAB W: AAAB),采用 kmp 我们可以将时间复杂度降低到 m + n。问题的关键就在如何利用已知不匹配时字符串的信息。我们带着这个问题来往下看。

我们可知此时 D 不匹配，并且除了 S[0] 其他的已遍历索引(S[1] S[2] S[3])都跟 W[0]不相同（根据部分匹配表），则直接跳过 S[1] - S[3]的匹配.这里提到了部分匹配表，我们下面说如何生成，我们先初步了解他的作用。

上面 D 不匹配 ，我们知道 D 前面是 ABC，明确可知 D 不匹配的时候因为 D 前面没有与 W 首部开始算起重复的字符，所以我们可以直接跳过 D 前面不匹配的字符，这里可能还有点不理解，我们继续往下看。

```
    index: 01234567890123456789012
        S: ABC ABCDAB ABCDABCDABDE
        W:     ABCDABD
        i:     0123456
```

此时，由上可知 W 中第二个字符 D 此时不匹配，这是我们可以知道第二个字符串前面字符为 ABCDAB，它前面的字符 AB 与开头的 AB 重复，此时我们可以利用这一点，跳过部分，下次匹配的时候直接从 W 的第三个索引下开始匹配，因为我们明确知道 AB 是相同的。由此:

```
    index: 01234567890123456789012
        S: ABC ABCDAB ABCDABCDABDE
        W:         ABCDABD
        i:         0123456
```

此时，C 匹配失败，跟第一步原因相同，我们跳过

```
    index: 01234567890123456789012
        S: ABC ABCDAB ABCDABCDABDE
        W:            ABCDABD
        i:            0123456
```

此时，末尾 D 匹配失败，跟之前 D 匹配失败同样的理由，我们跳过部分

```
    index: 01234567890123456789012
        S: ABC ABCDAB ABCDABCDABDE
        W:                ABCDABD
        i:                0123456
```

匹配完成。
此时，我们拿到了 S 是否含有 W 的结论，那么这个部分匹配表的求法，以及用法呢？？？

## <span name = "indextable">部分匹配表（next）</span>

> 我们根据过程可以知道这个[匹配表](/2021/kmp.md#bu-fen-pi-pei-biao-next),其实就是当前索引之前的字符是否含有与原字符串首部开始重复的字符。
> 很多教程在这里就是说真前缀 真后缀，说的对，但是很多都是复制粘贴回答......
>
> next 表也可以说是 i 之前的字符串中，有多大长度的相同真前缀后缀。
> 这里是所谓的真前缀就是 前缀中不包含自己，真后缀 后缀中不包含自己，这里需要查看前缀后缀的概念理解，随后补上。也就是说：
>
> 真前缀就是不等于原串的前缀，真后缀就是不等于原串的后缀。
>
> 还是以 ABCDABD 为例求

```
    ABCDABD
    0
```

首先我们索引到 0，发现 A 没有前缀，后缀，所以为 0。

```
    ABCDABD
    00001
```

B: 真前缀 A 真后缀 B 无

C: 真前缀 A AB 真后缀 C BC 相同无

D: 真前缀 A AB ABC 真后缀 D CD BCD

A: 真前缀 A AB ABC ABCD 真后缀 A DA CDA BCDA 相同 A 赋值为 1

```
    ABCDABD
    0000120
```

B: 真前缀 A AB ABC ABCD ABCDA 真后缀 B AB DAB CDAB BCDAB 相同 AB 赋值为 2

D: 真前缀 A AB ABC ABCD ABCDA ABCDAB 真后缀 D BD ABD DABD CDABD BCDABD 相同 无

综上 next 表为 [0,0,0,0,1,2,0]。到这里我们可以发现求得其实就是自己前面含有从首部开始的重复字符数量。很多人都是以-1 开头，这个我们后面解释，其实我觉得算作个人习惯吧。

[wiki](https://zh.wikipedia.org/wiki/KMP%E7%AE%97%E6%B3%95)里面有另一种计算方法，是判断真后缀与前缀，计算结果为[-1,0,0,0,0,1,2],这里的优点是使用 next 时我们可以直接使用当前索引，不需要去做特殊处理。我们这里不介绍。

## 代码实现

> 我们还是以上面的为测试用例。

-   首先求 next
    ```
        let W = ABCDABD;
        let len = w.length;
        let next = new Array(len).fill(0);
        let now = 0; // 当前重复索引
        let index = 1; // 当前索引
        while(index < len){
            if(W[now] === W[index]){
                // 当左边与右边存在重复时
                now++;
                next[index] = now; // 此索引下重复数量加1
                index++; // 更新索引
            } else if(now > 0){
                // 当不存在相等重复时,判断now是否大于0，大于则更新
                // eg ABABD 当检查D时now为2,W[2] !== W[4],此时我们下一步需要判断W[1]是否跟W[4]相同
                // 至于为什么这里会更新为next[now - 1]，是因为每次我们的判断都是从头开始。
                // eg AACDEAAA [0,1,0,0,0,1,2,2]
                now = next[now - 1];
            }else{
                // 此时now为0 不存在重复
                next[index] = 0;
                index++;
            }
        }
    ```
- 查找
```
    let W = ABCDABD;
    let S = "ABC ABCDAB ABCDABCDABDE";
    let pos = 0;
    let index = 0;
    while(index < S.length){
        if(S[index] === W[pos]){
            // 相等索引更新
            pos++;
            index++;
        }else if(pos !== 0){
            // 不相等，且此时匹配的数量不为0
            // 根据当前匹配位置，去next表查询重复数量，作为下次跳过索引。
            pos = next[pos - 1];
        }else{
            // 都不符合 更新索引
            i++;
        }

        if(pos === W.length){
           // 匹配结束了
           return
        }
    }
```