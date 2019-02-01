title: safari中window.open 被拦截
author: 丘山
tags:
  - JavaScript
  - 代码片段
categories:
  - 前端
date: 2019-02-01 15:00:00
---
> safari、以及某些浏览器会被安全拦截。通过异步的方式调用，可以解决

```
// open 方法返回一个对象：此时打开一个新tab
var windowLink = window.open('', '_blank');

// 触发资源下载
windowLink.location = openUrl;

// 关闭方法
windowLink.close();

```