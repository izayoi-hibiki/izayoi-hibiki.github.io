---
title: 记一下TypeScript配合React的技巧
categories:
  - Web前端/随笔/后端
date: 2020-06-05 10:46:56
tags:
---

记一下TypeScript配合React的技巧

<!--more-->

# 自己写的组件里用了 Antd 的 Form 组件
自己写的组件里用了 Antd 的 Form 组件时，自己的组件想通过 props 抛出个事件，可以写个 props 接口继承 FormProps 这个接口，然后在接口里定义事件函数签名，这样在使用这个组件时 props 里调用这个函数时就不会报错了。