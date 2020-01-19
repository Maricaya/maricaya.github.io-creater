---
title: "Js函数全解"
date: 2020-01-14T17:01:00+08:00
draft: true
---

## 函数是什么？
函数三要素：返回值、

### 函数返回值由什么决定？
- 调用时输入的参数params
- 定义时的环境env

#### 举个🌰

```js
let x = 'x'
let a = '1'
function f1(x){
  return x + a
};
a = '3'
{
  let a = '2'
  alert(f1('x')) // 值为多少
}
a = '4'
```
结果为：a3
说明：a是定义时的a，而不是执行时的a

另一个🌰
```js
let x = 'x'
let a = '1'
function f1(c){
  c()
};
{
  let a = '2'
  function f2(){
    console.log(x + a)
  }
  f1(f2)
}
```
结果为：a2
说明：a是定义时的a，而不是执行时的a

### 闭包：函数内部能够访问外部的变量
一个常见考题：
```js
for(var i=0; i<6; i++){
  setTimeout(
    ()=>console.log(i) // i
  )
}
```
答案：6个6，因为6个函数共用一个i

如何改造？打印1-6
1. 将var改为let

### 结论：
1. 闭包的特点
- 能让一个函数维持住一个变量
- 但并不能维持这个变量的值
- 尤其是变量的值会变化的时候