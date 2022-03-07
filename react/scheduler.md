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

  while (currentTask !== null && !enableSchedulerDebugging) {
    if (
      currentTask.expirationTime > currentTime &&
      (!hasTimeRemaining || shouldYieldToHost())
    ) {
      // This currentTask hasn't expired, and we've reached the deadline.
      // 这个currentTask还没有过期，而且我们已经到了最后期限。
      // 安排一个宏任务在当前事件处理完毕后运行。 （setImmediate or MessageChannel）
      break;
    }

    var callback = currentTask.callback;

    if (typeof callback === "function") {
      currentTask.callback = null;
      currentPriorityLevel = currentTask.priorityLevel;
      var didUserCallbackTimeout = currentTask.expirationTime <= currentTime;
      // 真正的执行回调
      var continuationCallback = callback(didUserCallbackTimeout);
      currentTime = exports.unstable_now();

      if (typeof continuationCallback === "function") {
        currentTask.callback = continuationCallback;
      } else {
        if (currentTask === peek(taskQueue)) {
          pop(taskQueue);
        }
      }

      advanceTimers(currentTime);
    } else {
      pop(taskQueue);
    }

    currentTask = peek(taskQueue);
  } // Return whether there's additional work

  if (currentTask !== null) {
    return true;
  } else {
    var firstTimer = peek(timerQueue);

    if (firstTimer !== null) {
      requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime);
    }

    return false;
  }
}
```
