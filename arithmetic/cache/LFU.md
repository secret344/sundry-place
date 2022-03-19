# LFU Least Frequently Used 最不经常使用

> 做 leetcode 432. 全 O(1) 的数据结构 了解到的东西，在此记录下。就是 432 的解题思想。

注意：这篇文章只是为了初步了解 LFU 的思想。优化将会放到以后，如果碰到类似实际案例，再回来继续写。

## 解释

> 如果一段数据在之前被访问的次数很少，那么将来被访问的概率也会很低。根据访问频次以及访问时间（同访问频次下）来淘汰数据。

不列举图片了简单数字模拟下。

- 假设一个访问顺序：1 3 3 1 4 5 4 4 （数据是我瞎打的）
- 缓存容量设置为 4

| <center>访问层数（访问了多少次）<center> | <center>序号<center> |
| :--------------------------------------: | :------------------: |
|                    1                     |          5           |
|                    2                     |         1 3          |
|                    3                     |          4           |

- 此时 缓存内容为 1 3 4 5 如果在访问一个新值那么就会溢出，此时我们需要删除一个。
- 根据上表可知，我们只需移除 5 即可。

## 优缺点

优缺点暂时写一点

### 优点

- 上图可以看出，一般情况下 LFU 效率高于 LRU，避免周期性 偶发性的操作导致缓存命中率下降

### 缺点

- 复杂度较高
- 新缓存不友好，如果 5 是新加入的，那么后面再来一个 6 则 5 会被清理掉
- 早期数据相比后期容易缓存起来

## 实现

> LFU 的实现有多种，我们就简单实现一种就行，思想都是相同的，选择的数据结构不同。
先搞一个简单的写法，没有去实现真正意义上的LFU。

```javascript
class LFULink {
  count;
  keys;
  prev;
  next;
  constructor(count, key) {
    this.count = count || 0;
    this.keys = new Set();
    key && this.keys.add(key);
  }
  insert(node) {
    // 保存root root.prev指向末尾节点
    // 只有尾节点插入才是这个作用
    // 非尾节点正常作用 更新next
    node.next = this.next;
    node.next.prev = node;

    this.next = node;
    node.prev = this;
    return node;
  }
  delete() {
    this.prev.next = this.next;
    this.next.prev = this.prev;
  }
}

class LFUInstance {
  root;
  nodes; // 记录访问过的数据 以及对应的链表节点
  constructor() {
    this.root = new LFULink();
    this.root.prev = this.root; // 哨兵，双向链表 prev始终指向链表末尾节点
    this.root.next = this.root;
    this.nodes = new Map();
  }
  // 访问一个资源 插入链表
  insert(key) {
    let cur = this.nodes.get(key);
    if (cur == null) {
      // 数据第一次访问
      if (this.root.next === this.root || this.root.next.count > 1) {
        // 此时是创建首个节点，或者链表不存在访问了一次的节点
        // 创建一个访问一次的链表节点,存入key
        this.nodes.set(key, this.root.insert(new LFULink(1, key)));
      } else {
        this.root.next.keys.add(key);
        this.nodes.set(key, this.root.next);
      }
    } else {
      // 不是首次访问
      if (cur.next === this.root || cur.next.count > cur.count + 1) {
        // 如果是末尾节点 或者后一位节点次数不是 cur.count + 1
        this.nodes.set(key, cur.insert(new LFULink(cur.count + 1, key)));
      } else {
        cur.next.keys.add(key);
        this.nodes.set(key, cur.next);
      }
      cur.keys.delete(key);
      if (cur.keys.size === 0) {
        cur.delete();
      }
    }
  }
  // 删除一次资源
  delete(key) {
    let node = this.nodes.get(key);
    if (node.count === 1) {
      // 只出现了一次 直接删除
      this.nodes.delete(key);
    } else {
      // 最少出现2次
      let prev = node.prev;
      if (prev === this.root || prev.count < node.count - 1) {
        // 是不是node节点后一位， 或者前一位不是他自身次数减1的节点
        this.nodes.set(key, prev.insert(new LFULink(node.count - 1, key)));
      } else {
        prev.keys.add(key);
        this.nodes.set(key, prev);
      }
    }
    node.keys.delete(key);
    if (node.keys.size === 0) {
      node.delete();
    }
  }
}
```



## 优化将会放到以后