---
title: Write a debounce function
date: 2019-08-12 19:51:10
categories:
- Javascript
tags:
---

Debounce refers to that a function executes only the last time no matter how many callbacks are triggered in a certain period of time.

The realization principle is to set a timer when the function is first executed. After that, it is found that the timer has been set before the timer is cleared, and a new timer is reset. If there is a timer that has not been emptied, the trigger function is executed when the timer is finished.

```Javascript
// fn 是需要防抖处理的函数
// wait 是时间间隔
function debounce(fn, wait = 50) {
    // 通过闭包缓存一个定时器 id
    let timer = null
    // 将 debounce 处理结果当作函数返回
    // 触发事件回调时执行这个返回函数
    return function(...args) {
        // this保存给context
        const context = this
       // 如果已经设定过定时器就清空上一次的定时器
        if (timer) clearTimeout(timer)
      
       // 开始设定一个新的定时器，定时器结束后执行传入的函数 fn
        timer = setTimeout(() => {
            fn.apply(context, args)
        }, wait)
    }
}

// DEMO
// 执行 debounce 函数返回新函数
const betterFn = debounce(() => console.log('fn 防抖执行了'), 1000)
// 停止滑动 1 秒后执行函数 () => console.log('fn 防抖执行了')
document.addEventListener('scroll', betterFn)
```

However, Debounce in underscore also has a third parameter: immediate. What is this parameter used for?

Pass true to the immediate parameter, Debounce will trigger the function before the wait time is calculated (that is, the function will be triggered without any delay), rather than after the wait time, and will not trigger within the wait time (equivalent to locking the execution of FN). If you accidentally click the submit button twice, the second submission will not be executed.


Then we decide how to execute FN according to the value of immediate. In the case of immediate, we execute FN immediately and lock the execution of FN within the wait time. FN will not be executed again until it is triggered after the wait time, and so on.

```Javascript
// immediate 表示第一次是否立即执行
function debounce(fn, wait = 50, immediate) {
    let timer = null
    return function(...args) {
        // this保存给context
        const context = this
        if (timer) clearTimeout(timer)
      
       // immediate 为 true 表示第一次触发后执行
       // timer 为空表示首次触发
        if (immediate && !timer) {
            fn.apply(context, args)
        }
       
        timer = setTimeout(() => {
            fn.apply(context, args)
        }, wait)
    }
}

// DEMO
// 执行 debounce 函数返回新函数
const betterFn = debounce(() => console.log('fn 防抖执行了'), 1000, true)
// 第一次触发 scroll 执行一次 fn，后续只有在停止滑动 1 秒后才执行函数 fn
document.addEventListener('scroll', betterFn)
```

**Debounce in underscore**

After reading the basic version of the code above, I feel quite relaxed. Now let's learn how to implement the debounce function in underscore, learn the excellent ideas, and directly write the code and comments. The source code analysis depends on underscore v1.9.1 implementation.

```Javascript
// 此处的三个参数上文都有解释
_.debounce = function(func, wait, immediate) {
  // timeout 表示定时器
  // result 表示 func 执行返回值
  var timeout, result;

  // 定时器计时结束后
  // 1、清空计时器，使之不影响下次连续事件的触发
  // 2、触发执行 func
  var later = function(context, args) {
    timeout = null;
    // if (args) 判断是为了过滤立即触发的
    // 关联在于 _.delay 和 restArguments
    if (args) result = func.apply(context, args);
  };

  // 将 debounce 处理结果当作函数返回
  var debounced = restArguments(function(args) {
    if (timeout) clearTimeout(timeout);
    if (immediate) {
      // 第一次触发后会设置 timeout，
      // 根据 timeout 是否为空可以判断是否是首次触发
      var callNow = !timeout;
      timeout = setTimeout(later, wait);
      if (callNow) result = func.apply(this, args);
    } else {
     // 设置定时器
      timeout = _.delay(later, wait, this, args);
    }

    return result;
  });

  // 新增 手动取消
  debounced.cancel = function() {
    clearTimeout(timeout);
    timeout = null;
  };

  return debounced;
};

// 根据给定的毫秒 wait 延迟执行函数 func
_.delay = restArguments(function(func, wait, args) {
  return setTimeout(function() {
    return func.apply(null, args);
  }, wait);
});
```

Compared with the basic version implementation above, underscore has the following functions.

- The result value result is returned after the execution of the func function
- After the timer is timed, clear timeout so that it does not affect the triggering of the next continuous event
- The manual cancel function is added
- If immediate is true, it will only be executed when the callback is triggered for the first time. It will not be executed after the callback is triggered frequently