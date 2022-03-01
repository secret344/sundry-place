# Rabin–Karp

> 是一种由理查德·卡普与迈克尔·拉宾于1987年提出的、使用散列函数以在文本中搜寻单个模式串的字符串搜索算法单次匹配。该算法先使用旋转哈希以快速筛出无法与给定串匹配的文本位置，此后对剩余位置能否成功匹配进行检验。此算法可推广到用于在文本搜寻单个模式串的所有匹配或在文本中搜寻多个模式串的匹配。


## 旋转哈希

举例说明:
   
s:   1 3 4 5 6 7 8 9 

p:   5 6 7

1. 首先计算 S[0] - S[2]（所要匹配字符串的长度）的哈希），我们直接使用数字代替为 134，那么所要匹配字符串p的hash就是567
2. 开始进行匹配,此时指针指向下标2，首先我们对比 S[0-2]  与 p 的hash,不匹配，此时指针往右移动一位。
3. 此时需要计算 S[1-3] 的hash,我们只需利用 S[0-2] 的hash减去S[0]加上S[3]即可。
    - 即为： (S[0-2] - (S[0] * 10^p.length-1)) * 10 + S[3]
    -  ( 134 - 1 * 10^2 ) * 10 + 5 = 345
4. 当hash匹配时，我们需要对字符串进行比对，一致则查找到。

### 简易表示

- hash[x] =(...((hash[x] * r + hash[x - 1]) * r + hash[x - 2]) * r ... + hash[0])
- 用 m 代表所要匹配的字符串位数
- 同 r 代表进制
- hash[x] 代表一个hash
- Hash[x] 代表单个位

> hash[x + 1] = ( hash[x] - Hash[x] * r^m-1 ) * r + Hash[x + m]
> 
> 粗略解释： 下一位hash等于之前hash减去 最高位 加上 后一位

### hash模运算

当只存在上述计算时，有可能会计算出很大的值，导致溢出，所以我们要对hash计算进行mod运算。
此时,计算方式修改为：

> hash[x] =(...(((((hash[x] * r) mod q + hash[x - 1]) * r)mod q + hash[x - 2]) * r)mod q ... + hash[0]...) mod q

1. 当我们在计算 最高位 的位次时发现，
  - Hash[x] = ((((((((Hash[x] mod q) * r) mod q) * r) mod q)*r)mod q) ... * r) mod q 
  - 根据 模运算的分配律 (a×b) mod c=(a mod c * b mod c) mod c 
  - (((((Hash[x] * r mod q) * r mod q)  *r  mod q) ...* r mod q) * r mod q 
  - 我们用 h ≡ r^m-1(% q)
  - Hash[x] = Hash[x] * h
2. 此时： hash[x + 1] = (( hash[x] - Hash[x] * h ) * r + Hash[x + m]) % q

### 根据以上可以实现算法

```JavaScript
    function RabinKarp(s, p){
        let
    }
```