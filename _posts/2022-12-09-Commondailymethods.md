---
layout: post
title: 'js常用方法'
date: 2023-6-9 15:05:50 +0800
subtitle: 'js常用方法'
tags: 方法 js
categories: 技术
---

## js 实现在找出一个数组中时间最大的那一项

> 可以使用 JavaScript 的  `Math.max()`  函数来实现这个目的。下面是一个简单的示例，假设你有一个名为  `dates`  的数组，里面存储了若干个日期对象：

```javascript
var dates = [
   new Date(2022, 9, 1),
   new Date(2022, 0, 1),
   new Date(2022, 11, 31),
   new Date(2022, 2, 1),
]
// 获取日期最大的项
var maxDate = Math.max.apply(null, dates)
```

> 你还可以使用 JavaScript 的  `Array.prototype.reduce()`  方法来实现同样的目的，下面是一个示例：

```js
var dates = [
   new Date(2022, 9, 1),
   new Date(2022, 0, 1),
   new Date(2022, 11, 31),
   new Date(2022, 2, 1),
]
// 获取日期最大的项
var maxDate = dates.reduce(function (a, b) {
   return a > b ? a : b
})
```

## js 监听页面的显示和隐藏

> 虽然 vue 是单页面应用，但当我们有需求要在项目中多开页面时候，如果此页面有较多需要渲染的图表时，通过监听 <mark style="background: #ADCCFFA6;">visibilitychange</mark> 可以得知当前页面的显示与隐藏，我们可以把隐藏的页面销毁等操作，用来释放被隐藏页面的内存。

```js
document.addEventListener('visibilitychange', e => {
   if (document.hidden) {
      this.$destroy(true) // 隐藏页面销毁组件
   } else {
      this.$router.go(0) // 显示页面重新刷新获取数据
   }
})
```
