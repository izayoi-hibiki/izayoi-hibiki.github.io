---
title: 'ReferenceError: Can''t find variable: globalThis'
categories:
  - Web前端
date: 2021-07-12 15:27:10
tags:
  - 兼容性
  - safari
---

> [https://github.com/es-shims/globalThis/issues/16](https://github.com/es-shims/globalThis/issues/16)

<!--more-->

## 原因
safari 12.1 以下版本不支持 globalThis，需要添加 polyfill

## 报错
```text
ReferenceError: Can't find variable: globalThis
```
## 解决方案

npm 安装 globalthis
```
npm i globalthis
```

在 angular 的 src/polyfill.ts 文件内添加 
```JavaScript
import 'globalthis/auto';
```

