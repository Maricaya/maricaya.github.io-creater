---
title: "EventLoop计时器nextTick"
date: 2020-01-09T20:17:00+08:00
draft: true
---

node 的event Loop
chrome

timer   执行 setTimeout 和 setInterval 的回调函数
poll    获取新的 I/O 事件。在某些场景下 Node.js 会阻塞在这个阶段。
check   执行 setImmediate() 的回调函数。

setImmediate 的主要优势就是，如果在 I/O 操作的回调里，setImmediate 的回调总是比setTimeout 的回调先执行。（译者注：怎么总是把一个道理翻来覆去地说）
process.nextTick() 当前阶段的后面

###总结
-   Chrome
    -   宏任务 马上（MarcoTask)
    -   微任务 一会（MicroTask）
-   Node
    -   setTimeOut timers
    -   setImmediate check
    -   nextTick    当前后面
        -   primise.then(fn) resolve()时，fn放入当前队列后面执行
        -   async await 转换为then

从一道面试题说 event loop
```js
(function () {
  setTimeout(() => {
    console.log(0)
  })
  new Promise(resolve => {
    console.log(1)
    setTimeout(() => {
      resolve()
      Promise.resolve().then(() => console.log(2))
      console.log(3)
    })

    Promise.resolve().then(() => {console.log(4)})
  }).then(() => {
    console.log(5)
    Promise.resolve().then(() => console.log(8))
    setTimeout(() => console.log(6))
  })
  console.log(7)
})()
```
1
7
4
0
3
5
2
8
6