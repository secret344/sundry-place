# 十大经典排序算法

## 附:[排序算法](https://zh.wikipedia.org/wiki/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95)

## 1.冒泡排序

> 重复比较要排序的数组相邻单位，顺序错误就互相交换位置。直到不再进行交换，表明排序结束。

```javascript
function sort(arr) {
    let len = arr.length;
    for (let i = 0; i < len; i++) {
        for (let j = 0; j < len; j++) {
            if (arr[j - 1] > arr[j]) {
                let temp = arr[j - 1];
                arr[j - 1] = arr[j];
                arr[j] = temp;
            }
        }
    }
    return arr;
}
```

## 2.选择排序

> 先找到最小（大）的元素，排到起始位置，然后在剩余元素中继续找最小（大）的元素，排到已排序的末尾。

```javascript
function sort(arr) {
    let len = arr.length;
    for (let i = 0; i < len; i++) {
        let minindex = i;
        for (let j = i + 1; j < len; j++) {
            if (arr[j] < arr[minindex]) {
                minindex = j;
            }
        }
        let temp = arr[minindex];
        arr[minindex] = arr[i];
        arr[i] = temp;
    }
    return arr;
}
```

## 3.插入排序

> 从第二位开始循环，每次都检查该位置之前所有小（大）于他的元素，直到找到不小（大）于他的元素，插入到该位置。

```javascript
function sort(arr) {
    let len = arr.length;
    for (let i = 1; i < len; i++) {
        let pre = i - 1;
        let current = arr[i];
        while (pre >= 0 && arr[pre] > current) {
            arr[pre + 1] = arr[pre];
            pre--;
        }
        arr[pre + 1] = current;
    }
    return arr;
}
```

## 4.希尔排序

> 插入排序改进版本，加入了步长这一概念。最终目的是，在步长为一时所做的插入排序比较所造成的移动次数较小，以此来优化性能。
>
> 步长：假设步长为 5，就是对比当前元素与 5 个单位之前的元素，如果符合交换条件，就交换位置。

```javascript
function sort(arr) {
    let len = arr.length;
    // 位运算来计算步长
    for (let gap = len >> 1; gap > 0; gap >>= 1) {
        for (let i = gap; i < len; i++) {
            let temp = arr[i];
            // 插入排序
            let j = i - gap;
            while (j >= 0 && arr[j] > temp) {
                arr[j + gap] = arr[j];
                j -= gap;
            }
            arr[j + gap] = temp;
        }
    }
    return arr;
}
```

## 5.归并排序

> 采用分治思想，把序列分为 n/2 的长度，对每一个单独部分继续进行分割，直到序列数为 1，最后在对整体进行排序。
> 每个小整体都由 left 与 right 两个部分组成，此时这两个部分都已经是排序好的状态，此时只需要对比数组头部（尾部）元素，
> 满足条件则取出放入排序好的整体后面.

-   递归法

    ```javascript
    function sort(arr) {
        function merge(left, right) {
            let result = [];
            while (left.length > 0 && right.length > 0) {
                if (left[0] < right[0]) {
                    result.push(left.shift());
                } else {
                    result.push(right.shift());
                }
            }
            // 保证不会遗漏
            return result.concat(left, right);
        }

        let len = arr.length;
        if (len <= 1) {
            return arr;
        }
        let mid = Math.floor(len / 2);
        let left = arr.slice(0, mid);
        let right = arr.slice(mid);
        return merge(sort(left), sort(right));
    }
    ```

## 6.快速排序

> 挑选基准值
>
> 分割，将所有比基准值小的放入一边，大的放入另外一边。
>
> 递归排序子序列.

> 示例,两种不同的分割方法。

```javascript
function sort(arr, baseIndex = 0) {
    let left = 0,
        right = arr.length;
    if (right - 1 <= left) {
        return arr;
    }
    let base = arr[baseIndex];
    // 将基准位置放入最后一项
    arr[baseIndex] = arr[right - 1];
    arr[right - 1] = base;
    // 当前小于基准值的位置。
    // 最终作为分割点。
    let sortIndex = left;
    for (let i = left; i < right - 1; i++) {
        let ele = arr[i];
        if (ele <= base) {
            // 小于等于基准位，则讲其与sortIndex位置的元素替换，sortIndex+1
            // sortIndex之前的位置保证全部小于等于基准位
            arr[i] = arr[sortIndex];
            arr[sortIndex] = ele;
            sortIndex = sortIndex + 1;
        }
    }
    // 将基准位与sortIndex位互换,保证左边小于等于它，右边大于它
    let temp = arr[sortIndex];
    arr[sortIndex] = arr[right - 1];
    arr[right - 1] = temp;
    return sort(arr.slice(0, sortIndex), baseIndex).concat(
        arr[sortIndex],
        sort(arr.slice(sortIndex + 1), baseIndex)
    );
}
```

```javascript
function sort(arr) {
    if (arr.length <= 1) {
        return arr;
    }
    let base = arr[0],
        left = 0,
        right = arr.length - 1;
    while (left <= right) {
        if (left === right) {
            // 当查找完毕，将基准值放入最终索引位（也就是分割点）。
            arr[0] = arr[left];
            arr[left] = base;
            break;
        }
        // 找到左侧大于基准值的项目
        while (left < right && arr[right] >= base) {
            right--;
        }
        // 找到右侧大于基准值的项目
        while (left < right && arr[left] <= base) {
            left++;
        }
        // 替换两个元素。
        // 将小于基准值的元素放到前面。继续查找满足条件项
        let temp = arr[left];
        arr[left] = arr[right];
        arr[right] = temp;
    }
    return quick(arr.slice(0, left))
        .concat(arr[left])
        .concat(quick(arr.slice(left + 1)));
}
```

## 7.堆排序

## 8.计数排序

> 找到数组中的最大元素和最小元素。新建数组 c
>
> 统计数组中每个元素值为 i 出现的次数，存入数组 c 的第 i 项。
>
> 对所有的计数累加（从 C 中的第一个元素开始，每一项和前一项相加）；
>
> 反向填充目标数组：将每个元素 i 放在新数组的第 C[i]项，每放一个元素就将 C[i]减去 1

> 不放代码了，比较简单，不适合范围很大的排序，需要大量时间和内存。但是，计数排序可以用在基数排序算法中，能够更有效的排序数据范围很大的数组。
> 下面的基数排序会用到计数排序。

## 9.桶排序

## 10.基数排序

> 原理是将整数按位数切割成不同的数字，然后按每个位数分别比较。
>
> 将所有待比较数值（正整数）统一为同样的数位长度，数位较短的数前面补零。然后，从最低位开始，依次进行一次排序。这样从最低位排序一直到最高位排序完成以后，数列就变成一个有序序列。
>
> 取到最大数，确定位数。
>
> 原数组开始从最低位组成基数（radix）数组。
>
> 对基数数组进行计数排序（计数排序适用于小范围数的特点）

```javascript
function sort(arr) {
    let mod = 10,
        divisor = 1; // 取出当前位,  (n % 10) / 1 然后取整就是个位。
    let counter = [];
    let maxDigit = (Math.max(...arr) + "").length;
    for (let i = 0; i < maxDigit; i++) {
        // 计数排序
        for (let j = 0; j < arr.length; j++) {
            let digit = parseInt((arr[j] % mod) / divisor);
            if (!counter[digit]) {
                counter[digit] = [];
            }
            // 将当前元素放入计数排序数组相应的下标中
            counter[digit].push(arr[j]);
        }
        let pos = 0; //重新排列的原数组下标
        for (let j = 0; j < counter.length; j++) {
            // 取出计数排序数组项，重新排列原数组。
            while (!!counter[j] && counter[j].length) {
                let curset = counter[j].shift();
                arr[pos++] = curset;
            }
        }
        mod *= 10;
        divisor *= 10;
    }
    return arr;
}
```
