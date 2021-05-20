# Timers 定时器

## 定时器的基本含义

1. 先贴一段 html 规范的[原文](https://html.spec.whatwg.org/multipage/timers-and-user-prompts.html#timers)定义
    > The setTimeout() and setInterval() methods allow authors to schedule timer-based callbacks.
    >
    > setTimeout()和 setInterval()方法允许 authors 安排基于定时器的回调。
2. 定时器的一些注意事项

-   Timers can be nested; after five such nested timers, however, the interval is forced to be at least four milliseconds.定时器可以嵌套；但在五个嵌套定时器之后，间隔时间就会被强制为至少四毫秒。
    -   任务的定时器嵌套层既用于嵌套调用 setTimeout()，也用于由 setInterval()创建的重复定时器。换句话说，它代表了这个算法的嵌套调用，而不是某个特定方法的调用。
-   This API does not guarantee that timers will run exactly on schedule. Delays due to CPU load, other tasks, etc, are to be expected.这个 API 并不保证定时器会完全按计划运行。由于 CPU 负载、其他任务等造成的延迟是可以预期的。

## 定时器

### 前言

> [WindowOrWorkerGlobalScope](https://html.spec.whatwg.org/multipage/webappapis.html#windoworworkerglobalscope)存在一个活动定时器列表([list of active timers.](#list-of-active-timers))，该列表中的每一个条目都会有一个数字标识，并且在实现 WindowOrWorkerGlobalScope mixin 生命周期内该数字在列表中必须是唯一的。

### clearTimeout() clearInterval()

> 清除定时器，为什么放在最前面呢？是因为他们篇幅最少。
>
> 从 Window 或 WorkerGlobalScope 的活动定时器列表中清除标识为 handle 的条目，如果当前活动定时器列表不存在这个 handle 对应的条目，则函数调用时什么也不做

-   有个注意点，这俩函数作用对象都是 Window 或 WorkerGlobalScope 的活动定时器列表，因为是从同一个列表中清除条目，所以方法是通用的，为了维护还是建议一一对应。

### setTimeout()

> setTimeout()方法必须返回定时器初始化步骤所返回的值，将方法的参数传给它们，将运行算法的方法所在的对象（Window 或 WorkerGlobalScope 对象）作为方法上下文，并将 repeat 标志设置为 false。

### setInterval()

> 与 setTimeout()一致，不同之处是将 repeat 标志设置为 true。

### [定时器初始化](https://html.spec.whatwg.org/multipage/timers-and-user-prompts.html#timer-initialisation-steps)

* 定时器会生成一个 ***task***,当这个task被执行时,第一步会检查这个handle是否已经在活动定时器列表中被清除，如果已经被清除就放弃执行，最后一步会查看repeat标志，如果是true，就会重新执行定时器初始化，并且使用相同的方法参数、相同的方法上下文以及repeat标志。并将之前的handle设置为handler，如果是false就会从活动定时器列表中删除。
* 初始化结束会被放入 ***[Queue a global task](https://html.spec.whatwg.org/multipage/webappapis.html#queue-a-global-task)***。