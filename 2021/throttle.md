# 节流函数

> 在日常工作中，我们经常会碰到点击某个按钮，或者滚动条滚动需要触发请求的需求，当我们不使用节流函数时，就会发生点击一次，请求一次，点击事件不断触发，请求也在频繁发出，这样既容易造成页面卡顿，也会 导致服务器资源的浪费，这时候就需要节流函数来解决问题。
> 所以就引申出节流函数的作用：
>
> 一定时间内，限制方法的触发次数。

## 实现

```javascript
function throttle(fn, delay) {
    let timer = null;
    return function () {
        if (!timer) {
            timer = setTimeout(() => {
                fn(...arguments);
                clearTimeout(timer);
            }, delay);
        }
    };
}
```

> 上面实现原理上是利用定时器，在一定延迟时间后，执行函数。但是很明显，每次执行都得等待定时器结束，首次点击触发函数需要等待定时器结束。

```javascript
function throttle(fn, delay) {
    let startTime = 0;
    return function () {
        let endTime = Date.now();
        let interval = endTime - startTime;
        if (delay - interval <= 0) {
            fn(...arguments);
            startTime = Date.now();
        }
    };
}
```

> 本次实现保留开始时间（第一次为 0），每次点击获取当前时间，得到当前点击时间减去上次触发时间（startTime）的结果 ，这个结果就是上次函数触发所经过的时间，此时我们只需要判断延迟时间是否大于间隔时间，当延迟时间减去间隔时间小于等于 0，就满足再次执行函数的条件。这次时间当第一次点击时会立即执行，但是缺点是最后一次点击，若间隔时间不满足延迟时间，则不会执行。

```javascript
function throttle(fn, delay) {
    let startTime = 0;
    let timer = null;
    return function () {
        let endTime = Date.now();
        let interval = endTime - startTime;
        let remainning = delay - interval;
        // 每次进来都清除上次定时器
        timer && clearTimeout(timer);
        // 当剩余时间小于等于0 立即执行函数
        // 大于0则创建新的定时器，延迟时间设为剩余时间执行，确保最后一次点击会执行
        if (remainning <= 0) {
            fn(...arguments);
            startTime = Date.now();
        } else {
            timer = setTimeout(() => {
                fn(...arguments);
                clearTimeout(timer);
            }, remainning);
        }
    };
}
```

> 利用定时器实现方式以及每次点击判断剩余时间实现方式的优点来保证首次和最后一次的执行，首次进来时剩余时间小于 0，立即执行并更新上次触发时间（startTime），第二次点击时判断剩余时间，当大于 0 时，证明此时不满足函数执行，则生成定时器保存此次触发事件，当再次点击时清除上次定时器并且判断剩余时间，再次执行函数或者生成定时器。

## 附言

> lodash 在实现节流函数利用到了 requestAnimationFrame,当不存在 requestAnimationFrame 时也是利用了 setTimeout。
