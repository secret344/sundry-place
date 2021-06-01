# 十大经典排序算法

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

## 7.堆排序

## 8.计数排序

## 9.桶排序

## 10.基数排序
