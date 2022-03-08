# React 的 scheduler

1. 从 react 源码可知，react 通过 ensureRootIsScheduled 将 fiber 树推入构建流程

```javascript
function ensureRootIsScheduled(root, currentTime) {
  var existingCallbackNode = root.callbackNode; // Check if any lanes are being starved by other work. If so, mark them as
  ...
    var schedulerPriorityLevel;
    // lanesToEventPriority 将lane优先级转化为 scheduler 的优先级
    switch (lanesToEventPriority(nextLanes)) {
      case DiscreteEventPriority:
        schedulerPriorityLevel = ImmediatePriority; // 1
        break;

      case ContinuousEventPriority:
        schedulerPriorityLevel = UserBlockingPriority; // 2
        break;

      case DefaultEventPriority:
        schedulerPriorityLevel = NormalPriority; // 3
        break;

      case IdleEventPriority:
        schedulerPriorityLevel = IdlePriority; // 5
        break;

      default:
        schedulerPriorityLevel = NormalPriority; // 3
        break;
    }
    // scheduleCallback 将react事件作为任务交给 scheduler 调度
    newCallbackNode = scheduleCallback(
      schedulerPriorityLevel,
      performConcurrentWorkOnRoot.bind(null, root)
    );
  }
  // commit阶段会重置为null NoLane
  root.callbackPriority = newCallbackPriority;
  root.callbackNode = newCallbackNode;
}

```

> 这里在交给 scheduler 调度时做了一层优先级转化，这是因为 react 和 scheduler 各有一套优先级机制。

2. 接下来查看 scheduleCallback 的代码，对应 scheduler 中的 unstable_scheduleCallback

```javascript
function unstable_scheduleCallback(priorityLevel, callback, options) {
  var currentTime = exports.unstable_now(); // 返回当前经过时间 （当前时间减去开始时间）
  var startTime; // 任务开始时间

  // 开始时间延迟
  if (typeof options === "object" && options !== null) {
    var delay = options.delay;

    if (typeof delay === "number" && delay > 0) {
      startTime = currentTime + delay;
    } else {
      startTime = currentTime;
    }
  } else {
    startTime = currentTime;
  }

  var timeout; // 任务延迟时间
  //  根据优先级 给予延迟时间 优先级越高（数值越小）延迟时间越小，反之越大
  switch (priorityLevel) {
    case ImmediatePriority:
      timeout = IMMEDIATE_PRIORITY_TIMEOUT;
      break;

    case UserBlockingPriority:
      timeout = USER_BLOCKING_PRIORITY_TIMEOUT;
      break;

    case IdlePriority:
      timeout = IDLE_PRIORITY_TIMEOUT;
      break;

    case LowPriority:
      timeout = LOW_PRIORITY_TIMEOUT;
      break;

    case NormalPriority:
    default:
      timeout = NORMAL_PRIORITY_TIMEOUT;
      break;
  }
  //  计算过期时间
  var expirationTime = startTime + timeout;
  //   将react的事件 创建成一个任务
  var newTask = {
    id: taskIdCounter++,
    callback: callback,
    priorityLevel: priorityLevel,
    startTime: startTime,
    expirationTime: expirationTime,
    sortIndex: -1,
  };

  if (startTime > currentTime) {
    // 开始时间比当前时间大 是一项延迟的任务。
    newTask.sortIndex = startTime;
    push(timerQueue, newTask);
    // 如果 taskQueue（存放过期任务，立即执行的任务） 是空的，
    // 且当前任务时timerQueue优先级最高的任务，或者是第一个任务
    // 意味着当前不存在需要立即执行的任务（或者说当前任务比延迟队列中所有任务startTime都要小（优先级最高））
    // 重新开始一个 requestHostTimeout 来检查延时任务队列是否存在过期任务
    if (peek(taskQueue) === null && newTask === peek(timerQueue)) {
      // 所有的任务都是延迟的，而这是延迟最早的任务。
      if (isHostTimeoutScheduled) {
        // 当已经存在一个requestHostTimeout执行后，取消掉上次的执行
        // 减少资源浪费
        cancelHostTimeout();
      } else {
        //  标记requestHostTimeout执行
        isHostTimeoutScheduled = true;
      }
      // 执行延时开始任务，会生成一个 setTimeout 事件，setTimeout过期之后会执行handleTimeout
      requestHostTimeout(handleTimeout, startTime - currentTime);
    }
  } else {
    // 当前任务不是延时开始任务
    // sortIndex 修改为过期时间  过期时间越小代表优先级越高（等待时间少）
    newTask.sortIndex = expirationTime;
    push(taskQueue, newTask);
    // wait until the next time we yield.
    // isPerformingWork 检查flushWork是否已经开始执行
    // isHostCallbackScheduled handleTimeout执行时如果taskQueue已经存在任务，会将该参数修改为 true，然后执行requestHostCallback
    // flushWork，会将该参数修改为 false
    // 判断是否已经开始调度任务，没有则创建一个调度者，存在则不做操作，上一个调度者会调度该任务
    if (!isHostCallbackScheduled && !isPerformingWork) {
      isHostCallbackScheduled = true;
      requestHostCallback(flushWork);
    }
  }

  return newTask;
}
```

> scheduleCallback 主要任务就是根据 react 事件创建一个 scheduler 任务，根据任务开始时间判断任务过期时间，未过期的任务会加入 timerQueue 中，将 starttime 作为排序依据，过期的任务会存放到 taskQueue，将 expirationTime 作为排序依据。当 taskQueue 不存在任务时，如果此时新添加任务是一个延时开始任务，那么将会调用 requestHostTimeout 生成一个 settimeout，将第一个任务延时时间作为间隔时间，到期后调用 handleTimeout。当加入的是非延时开始任务时，直接加入 taskQueue，并判断当前是否存在调度者正在进行任务调度，不存在创建，存在则跳过。 3. handleTimeout 最终还是调用 requestHostCallback，所以我们直接从 handleTimeout 开始看。

3. handleTimeout

```javascript
function handleTimeout(currentTime) {
  // 标记，当handleTimeout 执行时，再次进来延时开始任务会重新启用一个requestHostTimeout
  // 保证存在任务时会存在一个调度者
  isHostTimeoutScheduled = false;
  //  检查延迟任务，当延迟时间结束推送进taskQueue队列
  advanceTimers(currentTime);
  //  flushWork执行时 isHostCallbackScheduled 便会修改为 false
  if (!isHostCallbackScheduled) {
    // 拿到非延迟任务
    if (peek(taskQueue) !== null) {
      isHostCallbackScheduled = true;
      requestHostCallback(flushWork);
    } else {
      // 拿到延迟任务 重新执行定时器
      var firstTimer = peek(timerQueue);

      if (firstTimer !== null) {
        requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime);
      }
    }
  }
}
```

> handleTimeout 主要任务时检查延时开始任务是否已经开始，开始就将其放入 taskQueue。 这个操作是由 advanceTimers 完成的。

4. advanceTimers

```javascript
function advanceTimers(currentTime) {
  // Check for tasks that are no longer delayed and add them to the queue.
  // 检查不再延迟的任务，并将其添加到队列中。
  var timer = peek(timerQueue);

  while (timer !== null) {
    if (timer.callback === null) {
      // Timer was cancelled.  延时开始任务被取消了
      pop(timerQueue);
    } else if (timer.startTime <= currentTime) {
      // Timer fired. Transfer to the task queue. 延时开始任务启动。转移到任务队列中。
      pop(timerQueue); // 删除
      timer.sortIndex = timer.expirationTime; // 将已经开始的Timer Task的排序依据修改为过期时间
      push(taskQueue, timer);
    } else {
      // Remaining timers are pending.
      //   当前 timerQueue 不存在需要开始的任务
      return;
    }

    timer = peek(timerQueue);
  }
}
```

> advanceTimers 检查 timerQueue 中是否存在需要移入 taskQueue 的任务。针对已经开始的 timer 任务会将过期时间作为排序依据。然后执行：

```javascript
// 拿到非延迟任务 上面handleTimeout已有注释
if (peek(taskQueue) !== null) {
  isHostCallbackScheduled = true;
  requestHostCallback(flushWork);
}
```

5. requestHostCallback

```javascript
function requestHostCallback(callback) {
  // flushWork
  scheduledHostCallback = callback;
  // isMessageLoopRunning 消息循环执行标记 防止多次执行
  if (!isMessageLoopRunning) {
    isMessageLoopRunning = true;
    schedulePerformWorkUntilDeadline();
  }
}
```

> 真正执行的是 schedulePerformWorkUntilDeadline，首先查看它的初始化

```javascript
var schedulePerformWorkUntilDeadline;

if (typeof setImmediate === "function") {
  // Node.js and old IE.
  // There's a few reasons for why we prefer setImmediate.
  //
  // Unlike MessageChannel, it doesn't prevent a Node.js process from exiting.
  // (Even though this is a DOM fork of the Scheduler, you could get here
  // with a mix of Node.js 15+, which has a MessageChannel, and jsdom.)
  // https://github.com/facebook/react/issues/20756
  //
  // But also, it runs earlier which is the semantic we want.
  // If other browsers ever implement it, it's better to use it.
  // Although both of these would be inferior to native scheduling.
  //   上面解释了为什么优先使用 setImmediate
  //   因为 setImmediate 更早执行且不会阻止 nodejs 进程退出。
  schedulePerformWorkUntilDeadline = function () {
    setImmediate(performWorkUntilDeadline);
  };
} else {
  var channel = new MessageChannel();
  var port = channel.port2;
  channel.port1.onmessage = performWorkUntilDeadline;

  schedulePerformWorkUntilDeadline = function () {
    port.postMessage(null);
  };
}
```

> schedulePerformWorkUntilDeadline 函数主要创建一个调度者，去执行 performWorkUntilDeadline

6. performWorkUntilDeadline

```javascript
var performWorkUntilDeadline = function () {
  if (scheduledHostCallback !== null) {
    var currentTime = exports.unstable_now(); // 当前时间

    // yieldInterval 默认5，根据 fps 改变 ，计算公式 Math.floor(1000 / fps);
    // 标记最后时间，在执行任务调度时 shouldYieldToHost 会使用该值判断是否已经超过（一帧时间）执行时间
    deadline = currentTime + yieldInterval;
    // 存在剩余时间
    var hasTimeRemaining = true;
    // 存在更多工作，发生中断
    var hasMoreWork = true;

    try {
      // scheduledHostCallback 就是之前的 flushWork
      hasMoreWork = scheduledHostCallback(hasTimeRemaining, currentTime);
    } finally {
      if (hasMoreWork) {
        // If there's more work, schedule the next message event at the end
        // of the preceding one.
        // 当存在任务中断时，重新开始一个新的调度
        schedulePerformWorkUntilDeadline();
      } else {
        //  任务全部执行完毕，初始化状态
        isMessageLoopRunning = false;
        scheduledHostCallback = null;
      }
    }
  } else {
    isMessageLoopRunning = false;
  } // Yielding to the browser will give it a chance to paint, so we can
};
```

> performWorkUntilDeadline 主要任务是 将当前帧所能执行的任务时间赋给 deadline，当作最后期限。然后执行 flushWork 开启真正的 workloop。当 flushwork 结束时，检查 flushwork 执行状态，如果存在任务中断，则重新开启一个新的调度者，不存在则初始化状态。

7. flushwork

```javascript
function flushWork(hasTimeRemaining, initialTime) {
  //  保证存在任务时会存在一个调度者 非延时任务新建时需要，返回scheduleCallback查看该标记作用
  //  unstable_continueExecution
  isHostCallbackScheduled = false;
  // 存在 timerQueue 开启的requestHostTimeout 任务，取消掉，因为已经存在调度
  if (isHostTimeoutScheduled) {
    // We scheduled a timeout but it's no longer needed. Cancel it.
    // 我们安排了一个超时，但已经不需要了。取消它。
    isHostTimeoutScheduled = false;
    cancelHostTimeout();
  }
  //  保证存在任务时只存在一个调度者 返回scheduleCallback查看该标记作用
  //  unstable_continueExecution
  isPerformingWork = true;
  var previousPriorityLevel = currentPriorityLevel;

  try {
    if (enableProfiling) {
      try {
        return workLoop(hasTimeRemaining, initialTime);
      } catch (error) {
        if (currentTask !== null) {
          var currentTime = exports.unstable_now();
          markTaskErrored(currentTask, currentTime);
          currentTask.isQueued = false;
        }

        throw error;
      }
    } else {
      // No catch in prod code path.
      return workLoop(hasTimeRemaining, initialTime);
    }
  } finally {
    currentTask = null;
    currentPriorityLevel = previousPriorityLevel;
    isPerformingWork = false;
  }
}
```

> flushWork 真正执行的是 workLoop ，并且在结束时重置相关状态。flushWork 返回 workLoop 的返回值

8. workLoop 真正的工作循环

```javascript
function workLoop(hasTimeRemaining, initialTime) {
  // 当前开始时间，赋值在 performWorkUntilDeadline 中
  var currentTime = initialTime;
  //  检查 timerQueue 是否存在开始的timerTask,将其放入 taskQueue中
  advanceTimers(currentTime);

  //  取出首个taskQueue（优先级最高）
  currentTask = peek(taskQueue);
  // 忽略enableSchedulerDebugging
  // 存在 currentTask 时进入循环
  while (currentTask !== null && !enableSchedulerDebugging) {
    // 判断 currentTask 过期时间是否大于当前时间
    // 过期时间大于当前时间代表当前任务还没过期
    // hasTimeRemaining 存在剩余时间（默认true）
    // shouldYieldToHost 是否中端当前任务，判断当前时间是否大于deadline（ currentTime + yieldInterval，去performWorkUntilDeadline查看该值）
    // 当currentTask未过期，但是当前执行时间已经大于等于deadline时（已达到最后期限），停止执行，发生中断
    if (
      currentTask.expirationTime > currentTime &&
      (!hasTimeRemaining || shouldYieldToHost())
    ) {
      // This currentTask hasn't expired, and we've reached the deadline.
      // 这个currentTask还没有过期，而且我们已经到了最后期限。
      // 安排一个宏任务在当前事件处理完毕后运行。 （setImmediate or MessageChannel）
      break;
    }
    // 拿到任务需要执行的函数
    var callback = currentTask.callback;
    // 判断 callback
    // 在 scheduleCallback 时会将 task 返回，此时react可以获取到task，然后赋值给 root.callbackNode
    if (typeof callback === "function") {
      currentTask.callback = null;
      // 保存当前任务的优先级 unstable_getCurrentPriorityLevel 可以获取
      currentPriorityLevel = currentTask.priorityLevel;
      // 当前任务是否过期
      var didUserCallbackTimeout = currentTask.expirationTime <= currentTime;
      // 真正的执行回调
      // continuationCallback 如果发生中断会返回一个函数
      var continuationCallback = callback(didUserCallbackTimeout);
      // 更新当前时间
      currentTime = exports.unstable_now();

      if (typeof continuationCallback === "function") {
        // 发生中断 让出执行权给优先级最高的任务
        currentTask.callback = continuationCallback;
      } else {
        // 执行结束 删除执行完的任务
        // 只有 taskQueue 首个任务是当前任务才会被删除，防止误删新的高优先级任务
        if (currentTask === peek(taskQueue)) {
          pop(taskQueue);
        }
      }
      // 检查 timerQueue 是否存在开始的任务 放入taskQueue中，
      // workLoop 在调度时会执行该任务
      advanceTimers(currentTime);
    } else {
      // 当前 callback 不是一个函数，直接取出
      // 取消 task 会将 callback 赋值为 null
      pop(taskQueue);
    }
    //  任务执行完，更新 currentTask 为新的任务
    currentTask = peek(taskQueue);
  } // Return whether there's additional work

  // workloop执行完毕 检查currentTask是否为null，不为null代表taskQueue还有任务未执行
  // 返回true 重新开启一个调度(会赋值给 performWorkUntilDeadline 的 hasMoreWork)
  if (currentTask !== null) {
    return true;
  } else {
    // 执行结束，检查 timerQueue 是否存在任务
    // 存在则开启一个 requestHostTimeout 在一定时间后 检查延时开始任务
    var firstTimer = peek(timerQueue);

    if (firstTimer !== null) {
      requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime);
    }

    return false;
  }
}
```

> workLoop 开始就是进行真正的任务调度，它分为 3 部分

- 首先检查 timerQueue 是否有可以放入 taskQueue 的任务。
- 开始进行 任务调度
- while 循环判断 taskQueue 的第一个任务（优先级最高）,如果存在，继续判断当前是否需要执行 task

  ```javascript
   if (
      currentTask.expirationTime > currentTime &&
      (!hasTimeRemaining || shouldYieldToHost())
    ) {
      break;
    }
  ```

  > 当前帧执行时间已经到达最后期限 且 当前 taskQueue 里面的任务还未到过期时间，终止当前 workloop, 下一个事件循环开始调度。

- 拿到要执行的 callback （真正需要执行的函数）
- 判断当前任务（callback）是否是一个函数，不是直接 pop 掉该任务，已经被取消掉了。
  - 是一个函数，保存当前优先级 currentPriorityLevel（调度结束会恢复默认（NormalPriority）），开始进行调度。
  - 判断 callback 的返回值
    - 判断 callback 的返回值，如果是一个函数，表明任务未执行完毕，将当前任务的 callback 替换为该函数
    - 如果 callback 的返回值不是一个函数，表明任务执行结束，判断 当前任务是否是 taskQueue 头，是则 pop
  - 然后执行 advanceTimers，完毕后本次循环结束。
- 执行结束，判断 currentTask 是否为 null
  - 是 null 表明当前 taskQueue 已经被清空，判断 timerQueue 是否为空，不为空则开启下次 requestHostTimeout。然后返回 false
  - 不是 null 表明存在中断，返回 true
- workloop 执行结束

9. workLoop 结束后，返回。

   - 此时回到 flushWork，然后回到 performWorkUntilDeadline
   - 当返回值为 true 时，hasMoreWork 为 true
     - 执行 schedulePerformWorkUntilDeadline（->performWorkUntilDeadline->scheduledHostCallback(flushWork)） 开启下次
   - 为 false，重置状态，结束。

10. callback 任务执行

- 此时我们回到 workLoop ，在执行 callback 会传入 didUserCallbackTimeout（任务是否过期）
- 此时 callback 就是我们开始所说，react 执行 scheduleCallback 传入的第二个参数

  ```JavaScript
  ...
    newCallbackNode = scheduleCallback(
    schedulerPriorityLevel,
    performConcurrentWorkOnRoot.bind(null, root)
      );
  }

  root.callbackPriority = newCallbackPriority;
  root.callbackNode = newCallbackNode;
  ```

  - scheduleCallback 生成的任务会交给 root.callbackNode
  - 可见，执行 callback 其实就是执行 performConcurrentWorkOnRoot

11. performConcurrentWorkOnRoot

```javascript
  function shouldTimeSlice(root, lanes) {
    if ((lanes & root.expiredLanes) !== NoLanes) {
      // 已经存在过期lane,为了防止更多饥饿任务
      return false;
    }
    // 不存在过期 判断是否含有 默认同步lane
    // 连续触发优先级，例如：滚动事件，拖动事件等 InputContinuousHydrationLane InputContinuousLane
    // 默认优先级，例如使用setTimeout，请求数据返回等造成的更新 DefaultHydrationLane DefaultLane
    var SyncDefaultLanes = InputContinuousHydrationLane | InputContinuousLane | DefaultHydrationLane | DefaultLane;
    return (lanes & SyncDefaultLanes) === NoLanes;
  }
  function performConcurrentWorkOnRoot(root, didTimeout) {
    // event time. The next update will compute a new event time.
    currentEventTime = NoTimestamp;
    currentEventTransitionLane = NoLanes;
    ...
    // didTimeout ： currentTask.expirationTime <= currentTime 未过期 false 过期 true
    // 根据条件执行 renderRootConcurrent 以及 renderRootSync
    // 任务过期或者不存在时间切片 同步执行 未过期且存在时间切片 并发
    // 返回值为 退出状态码
    var exitStatus = shouldTimeSlice(root, lanes) && ( !didTimeout) ? renderRootConcurrent(root, lanes) : renderRootSync(root, lanes);

    ...

    // 没有执行到commit阶段 就发生中断
    if (root.callbackNode === originalCallbackNode) {
      return performConcurrentWorkOnRoot.bind(null, root);
    }
    // 执行完毕
    return null;
  }
```

> performConcurrentWorkOnRoot 会根据条件判断然后去执行 renderRootConcurrent 或者 renderRootSync，因为 renderRootSync 是同步任务，不会被中断，我们接下来只看 renderRootConcurrent

12. renderRootConcurrent 我们只看有关 scheduler 的代码

```JavaScript
function renderRootConcurrent(root, lanes) {
  ...
  do {
    try {
      workLoopConcurrent();
      break;
    } catch (thrownValue) {
      handleError(root, thrownValue);
    }
  } while (true);
  ...
}
```

> 内部调用 workLoopConcurrent

13. workLoopConcurrent

```javascript
function workLoopConcurrent() {
  // Perform work until Scheduler asks us to yield
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}
```

> 该函数在循环创建 workInProgress 树，在每个单元执行完会判断 shouldYield 是否需要中断，当需要中断时，会停止循环。之前我们知道在执行commit阶段会重置 root.callbackNode = null,而且 commit 是不能被终端的。那么 root.callbackNode === originalCallbackNode，performConcurrentWorkOnRoot 函数就会返回 performConcurrentWorkOnRoot 作为 continuationCallback 的值。然后让出执行权。

此时，workLoop执行结束，如果存在终端，将 continuationCallback 赋值给 currentTask.callback，接着执行一次advanceTimers，然后取出最高优先级任务继续下次循环，如果执行时间过长导致的中断，跳出while。

跳出while之后会判断 currentTask 是否为 null，不为null表示还有任务未完成，返回 true，如果为 null，判断 timerQueue 是否存在任务，存在则调用requestHostTimeout等待间隔时间后开启下次调度，然后返回false。

此时，workloop结束，函数返回到 flushWork，flushWork会返回 workloop的返回值，然后重置状态。

flushWork结束， 返回到 performWorkUntilDeadline ，将上一步的返回值赋给 hasMoreWork，当hasMoreWork为true时，执行 schedulePerformWorkUntilDeadline 进行下次任务的调度。false则表示任务执行完毕，重置状态。结束。

此时我们就完成了对 react scheduler的探索。

## 总结：
    
react scheduler 核心就是使用小根堆维护 两个任务队列 timerQueue 以及 taskQueue，每次调度都是循环 taskQueue 进行执行，在适当时候使用 advanceTimers 将 timerQueue 放入 taskQueue 中进行调度。

scheduler 创建调度器时，优先使用 setImmediate 次要使用 MessageChannel，因为 MessageChannel 在nodejs环境下会阻止nodejs进程的退出，而 setImmediate 不会。
