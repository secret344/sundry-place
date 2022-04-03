# 前缀和

> 前缀和就是某下标之前所有元素之和（包括自身）

## 一维前缀和

> 公式: sum[i] = sum[i - 1] + num[i]

### 示例：

eg:

- 和为 K 的子数组

  - 解法 1

            ```javascript
                const subarraySum = (arr, target) => {
                // 思路
                // 求得和为 target 则需要求得每一个区间总和，此时需要求出前缀和，然后在判断某一个[0,n]区间所有子数组的和是否为target
                let ans = 0
                let prefix = []
                for (let i = 0; i < arr.length; i++) {
                    if (i === 0) prefix[i] = arr[i]
                    else prefix[i] = prefix[i - 1] + arr[i]
                }
                // 枚举当前区间内所有区间和是否等于target
                for (let i = 0; i < arr.length; i++) {
                    for (let j = 0; j <= i; j++) {
                        let cur = j === 0 ? prefix[i] : prefix[i] - prefix[j - 1]
                        if (cur === target) {
                            ans++
                        }
                    }
                }
                return ans
            }
            ```
        > 统计前缀和，然后根据前缀和进行遍历查找每一个区间大小，然后统计结果

    - 解法 2

            ```javascript
            const subarraySumHash = (arr, target) => {
                // 思路
                // 求得和为 target 则需要求得每一个区间总和，此时需要求出前缀和，然后在判断某一个[0,n]区间所有子数组的和是否为target
                let ans = 0
                let hash = {}
                let prefix = 0
                for (let i = 0; i < arr.length; i++) {
                    prefix += arr[i]
                    // 找到当前前缀和 减去 target 的前缀和sum
                    const key = prefix - target
                    // 存在多少个前缀和 即表明子数组存在多少个符合条件的结果
                    if (hash[key]) {
                        ans += hash[key]
                    }
                    // 判断当前前缀是否等于 target
                    if (prefix === target) {
                        ans++
                    }
                    // 统计所有前缀和的数量
                    if (!hash[prefix]) {
                        hash[prefix] = 1
                    } else {
                        hash[prefix] = hash[prefix] + 1
                    }
                }
                return ans

            }

            ```
        > 利用hash统计每一个前缀和的数量，每次在计算前缀和时，计算与target的差值（当前前缀和减去这个差值就是一个满足条件的区间），然后利用hash
        判断 该元素之前该前缀和区间的数量，结果加上该前缀和数量即可，然后需要判断该前缀和是否满足条件需要将 ans + 1 。返回最终的结果。

## 二维前缀和
