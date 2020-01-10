---
title: "实现Promise"
date: 2020-01-10T09:45:20+08:00
draft: true
---

- 该技术要解决什么问题 - why
  - 回调地狱
- 如何解决的 - how
- 什么优点（对比其他技术）
  - 减少缩进
  - 消灭if(err)
- 缺点
- 如何解决这些缺点

## Promise 的完整API是什么
### promise是一个类
- js中类是特殊的函数
- 类属性：length（可忽略）
- 类方法：all/...
- 对象属性: **then**/finally/catch
- 对象内部属性: **state** = pending/fulfilled/rejected