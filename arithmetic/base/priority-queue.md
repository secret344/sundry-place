# 优先队列

> 昨天被问到这个了，所以开一个文件，等交接之后搞一下小根堆，之前也了解过 react 的调度器，倒是没忘优先队列去想。昨天面试问的时候还感觉挺熟悉，但就是想不起来。

<!-- 2022年4月1日 -->

## 介绍

> 我感觉没什么介绍的，拿出维基百科的定义: 优先队列中的每个元素都有各自的优先级，优先级最高的元素最先得到服务；优先级相同的元素按照其在优先队列中的顺序得到服务。优先队列往往用堆来实现.

> react 的调度器就是采用小根堆的方式去实现的，我们在这就直接也实现一个小根堆吧。

## 实现

```javascript
class PriorityQueue {
  queue = [];
  constructor(compare = (a, b) => a - b) {
    this.compare = compare;
  }
  pop() {
    // 取出当前优先级最高的节点
    // 跟红黑树一样，如果我们直接取出头顶元素 然后对
    // 子节点进行对比移动的话 就会产生很多多余的判断
    // 所以我们直接用末尾节点替换，然后进行下移
    if (this.queue.length === 0) {
      return null;
    }
    let first = this.queue[0];
    let last = this.queue.pop();
    // 相等代表此时只存在一个节点 不需要调整
    if (last != first) {
      this.queue[0] = last;
      this.siftDown(0);
    }
    return first;
  }
  push(node) {
    this.queue.push(node);
    // 插入后进行上移
    this.siftUp(this.queue.length - 1, node);
  }
  siftUp(index, node) {
    // 找到父节点 进行对比 直到根节点
    //         0
    //       1   2
    //      3 4 5 6
    // 树的节点下一层为上一层的双倍
    // parent * 2 + 1 = child
    // eg: 0 * 2 + 1 = 1;
    // 1 * 2 + 1 = 3; 2 * 2 + 1 = 5
    // 所以 parent = (child - 1) / 2 向下取整即可
    while (index > 0) {
      const parentIndex = (index - 1) >>> 1;
      const parent = this.queue[parentIndex];
      if (this.compare(parent, node) > 0) {
        // 小根堆 如果 父节点大于当前节点 进行交换
        // 然后继续让父节点作为当前节点进行上移
        this.queue[parentIndex] = node;
        this.queue[index] = parent;
        index = parentIndex;
      } else return;
    }
  }
  siftDown(index) {
    let node = this.queue[index];
    let length = this.queue.length;
    // 向下调整 需要判断子节点当前最小的值 与父节点进行替换

    // 我们只需要保证 index < 当前长度的一半（当刚好大于时，
    // 表示接下来将会进入树的最后一层进行判断）
    let halfLength = length >>> 1;

    while (index < halfLength) {
      let leftIndex = (index << 1) + 1;
      let left = this.queue[leftIndex];
      let rightIndex = leftIndex + 1;
      let right = this.queue[rightIndex];

      // 进行查找
      if (this.compare(left, node) < 0) {
        // 小于左边 就需要判断是否小于右边
        // 当然我们也可以先找到 左右两边小的哪个
        if (rightIndex < length && this.compare(right, left) < 0) {
          // 如果右边小于左边
          this.queue[rightIndex] = node;
          this.queue[index] = right;
          index = rightIndex;
        } else {
          this.queue[leftIndex] = node;
          this.queue[index] = left;
          index = leftIndex;
        }
      } else if (rightIndex < length && this.compare(right, node) < 0) {
        this.queue[rightIndex] = node;
        this.queue[index] = right;
        index = rightIndex;
      } else return;
    }
  }
}
```


## 总结

小根堆使用 队列 来实现还是比较简单的，需要注意的也就是 父子索引的取法，以及交换的条件。