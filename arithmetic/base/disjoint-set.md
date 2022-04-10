# 并查集 Disjoint-set data structure

> 不相交数据结构，用于处理一些不交集的合并和查询问题。

## 初始化

```javascript
let fa = [];
function markSet(size) {
  for (let i = 0; i < size; i++) {
    fa[i] = i;
  }
}
```

## 查询

> 查询某个元素属于哪个集合，通常是返回一个“代表元素”。判断两个元素是否在同一个集合中。

```javascript
// 递归返回根
function find(x) {
  if (fa[x] === x) {
    return x;
  } else {
    return find(fa[x]);
  }

  // while (fa[x] !== x) {
  //     x = fa[x]
  // }
  // return x
}
```

## 合并

> 将两个集合合并成为一个

```javascript
// 合并就是将两个集合合二为一
// 只需要将某一个根节点指向另外一个就行
function merge(x, y) {
  fa[find(x)] = find(y);
}
```

## 路径压缩

> 显然 查询操作在最坏情况下是 On ,我们需要进行路径压缩,压缩放在查询阶段 我们将每一个节点都指向根节点，后续查找操作便省事很多

```javascript
function find(x) {
  if (fa[x] === x) {
    return x;
  } else {
    // 指向根节点
    fa[x] = find(fa[x]);
    return fa[x];
  }
}
```

## 按秩合并

> 我们只在查找阶段进行路径压缩，所以有时候并查集结构还是会比较复杂（比如我就合并不查询），这时候当 一个集合层级 大于另外一个 进行合并时，显然低层级根节点指向高层级根节点比较好，高指向低会导致总层级+1，多了不必要的查询次数。此时我们可以按秩合并，我们定义一个 rank 数组，保存根节点所对应的树的深度，合并时将小的合并到大的

> 执行路径压缩之后会改变树高度，此时我们没有更新 rank，所以会导致 rank 不准确。这里复制知乎 [Pin Chen](https://www.zhihu.com/people/pinchenjohnny)评论：秩不是准确的子树高，而是子树高的上界，因为路径压缩可能改变子树高。还可以将秩定义成子树节点数，因为节点多的树倾向更高。无论将秩定义成子树高上界，还是子树节点数，按秩合并都是尝试合出最矮的树，并不保证一定最矮。

```javascript
let rank = [];
function markSet(size) {
  for (let i = 0; i < size; i++) {
    fa[i] = i;
    rank[i] = 1;
  }
}
function merge(x, y) {
  let x = find(x),
    y = find(y);
  if (rnak[x] > rank[y]) {
    fa[y] = fa[x];
  } else {
    fa[x] = fa[y];
  }
  // 当rank相等时，我们需要将更新合并后的rank + 1
  if (rank[x] === rank[y] && x != y) {
    rabk[y] += 1;
  }
}
```

## 更多

> 简单的并查集介绍就这么多，很多经典题目或者[并查集应用](https://oi-wiki.org/topic/dsu-app/)就得等日后了，但是归根结底，理还是那么个理，慢慢积累吧。

## 参考

[算法学习笔记(1) : 并查集](https://zhuanlan.zhihu.com/p/93647900)

[并查集](https://oi-wiki.org/ds/dsu/#_10)
