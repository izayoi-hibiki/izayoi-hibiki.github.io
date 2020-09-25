---
title: Promise嵌套then和多级then执行顺序
date: 2020-09-24 15:14:38
categories:
    - Web前端
tags:
    - JavaScript
    - Promise
---

以一道例题分析 Promise 嵌套 then 和多级 then 执行顺序，例题来自简书上的[关于 Promise 嵌套 then 和多级 then 的解析](https://www.jianshu.com/p/b1abaf792491)，原作者讲的比较简洁，我稍微扩展下。

<!--more-->

# 例题：

```
Promise.resolve()
    .then(function F1() {
        console.log('promise1');
        Promise.resolve()
            .then(function F4() {
                console.log('promise2');
                Promise.resolve()
                    .then(function F5() {
                        console.log('promise4');
                    })
                    .then(function F6() {
                        console.log('promise7');
                    });
            })
            .then(function F7() {
                console.log('promise5');
            });
    })
    .then(function F2() {
        console.log('promise3');
    })
    .then(function F3() {
        console.log('promise6');
    });

setTimeout(function F0(){
    console.log('end');
}, 0);
console.log('start');
```

# 执行结果：

```
start
promise1
promise2
promise3
promise4
promise5
promise6
promise7
end
```

# 执行过程分析：

Promise 的执行过程只要牢记以下几点还是很好分析的：

1. JS 的 事件队列执行流程是 宏任务->所有微任务->UI Render->下一个宏任务->微任务->...
   宏任务包括同步代码、UI Render、I/O、setTimeout、setTimeInterval。
   微任务包括 Promise、Node 的 process.nextTick、Mutation Observer。
2. promise.then(fn1).then(fn2), then(fn1) 参数里的回调函数 fn1 执行时，fn1 的同步代码执行完毕这个 then(fn1) 就 resolve 了，会把后一个 then 里的参数回调函数放入微任务队列，这里是 fn2 (reject 了也一样，会把 reject 对应的回调放入微任务队列，这里只讨论 resolve 的情形)，链式 then 前一个 then resolve 之后才会把后一个 then 里的回调放入微任务队列。
3. `promise.then()`是异步的，`Promise.resolve()`是同步的。
4. 清空微任务队列后才会进入下一个宏任务。

所以，很容易就能理解为什么第一个输出 `start`，最后一个输出 `end`；因为 `console.log('start');`是同步代码，是宏任务，Promise 的代码是微任务，当前宏任务执行完之后才执行；setTimeout 是宏任务，代码执行到 setTimeout 时会将它的回调函数放入下一轮的宏任务队列中，所以当当前的宏任务（同步代码）和微任务（Promise）都执行完毕才执行 setTimeout 里的代码（下一轮宏任务）。这些比较好理解，我们重点分析 Promise 嵌套 then 和多级 then 的执行顺序。

分析这道题，我们不妨画一下当前的执行栈和微任务队列：

{% asset_img pic1.jpg  %}

1. 执行代码前，各队列还是空的状态。首先执行同步代码，`Promise.resolve()` 返回了一个 resolved 的 Promise 对象，执行后面的 then，then 接收到了 resolved，于是将 then 中的对应的异步代码回调函数 F1 放入微任务队列，然后执行下一句同步代码 setTimeout，setTimeout 把它的回调 F0 放到了宏任务队列，执行下一句同步代码`console.log('start');`输出了`start`。

{% asset_img pic2.jpg  %}

2. 同步代码执行完毕，执行栈清空，微任务队列非空于是按顺序执行微任务 F1,输出 promise1，又遇到了`Promise.resolve().then()`，于是将其回调 F4 放入微任务队列，至此 F1 函数的同步部分执行完毕，参数为 F1 的 then 返回了一个 resolving Promise 给了下一个 then，下一个 then 将 F2 放入微任务队列

{% asset_img pic3.jpg  %}

3. 同步代码执行完毕，执行栈清空，微任务队列非空于是按顺序执行微任务 F4，输出`promise2`，又遇到了`Promise.resolve().then()`，于是将其回调 F5 放入微任务队列，至此 F4 函数的同步部分执行完毕，参数为 F4 的 then 返回了一个 resolving Promise 给了下一个 then，下一个 then 将 F7 放入微任务队列

{% asset_img pic4.jpg  %}

4. 同步代码执行完毕，执行栈清空，微任务队列非空于是按顺序执行微任务 F2，输出`promise3`，执行完毕， then(F2) 返回了一个 resolving Promise 给了下一个 then(F3)，then(F3) 将 F3 放入微任务队列

{% asset_img pic5.jpg  %}

5. 同步代码执行完毕，执行栈清空，微任务队列非空于是按顺序执行微任务 F5，输出`promise4`，执行完毕，then(F5) 返回了一个 resolving Promise 给了 then(F6)，then(F6) 将 F6 放入微任务队列

{% asset_img pic6.jpg  %}

6. 同步代码执行完毕，执行栈清空，微任务队列非空于是按顺序执行微任务 F7，输出`promise5`，执行完毕，按顺序执行下一个微任务 F3，输出`promise6`，执行完毕，按顺序执行下一个微任务 F6，输出`promise7`。

{% asset_img pic7.jpg  %}

7.  F6 执行完毕，执行栈为空，微任务队列也是空的执行下一个宏任务 F0，输出`end`，执行完毕。

{% asset_img pic8.jpg  %}

# 小结

为了弄懂 Promise，也查了很多文章来看，提到`Promise.resolve()`时总能看到一句话：**“需要注意的是，立即 resolve()的 Promise 对象，是在本轮“事件循环”（event loop）的结束时执行，而不是在下一轮“事件循环”的开始时。”**。
贴的示例代码是下面这个：

```
setTimeout(function () {
  console.log('three');
}, 0);

Promise.resolve().then(function () {
  console.log('two');
});

console.log('one');
```

给我整的一头雾水：因为任务队列的执行顺序，微任务本来就在本次宏任务执行完毕之后执行啊，干嘛还要强调一遍？费了点劲，发现这些文章八成都是摘自大佬阮一峰写的[ECMAScript 6 入门](https://es6.ruanyifeng.com/#docs/promise#Promise-resolve)，示例代码也是一样的。评论区里有人 5 个月前就对此表示了跟我相同的疑问，阮大贴了个链接出来：[Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)，看完才明白想表达什么意思。个人感觉那段代码确实说明不了问题，但是要讲明白需要一篇文章才可以，在《ECMAScript 6 入门》里“Promise.resolve()”一小节里贴一大段文章确实也不太合适。而且原作者最终的结论与上文所述也略有不同，原文如下：

> -   Tasks execute in order, and the browser may render between them
> -   Microtasks execute in order, and are executed:
>     -   after every callback, as long as no other JavaScript is mid-execution
>     -   at the end of each task

-   宏任务按顺序执行，浏览器会在它们中间进行渲染。
-   微任务按顺序执行，并且在以下两种情况时被执行：
    -   在每个回调之后，只要没有任何 JS 代码正在执行
    -   在每个宏任务末尾

**[Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)** 这篇博文就是拿浏览器的冒泡事件来讲解的，强烈建议阅读这篇文章，作者不光举例恰当，还特意做了可以单步运行的动画来辅助理解，这要再看不懂那看什么文章也没用了(笑
