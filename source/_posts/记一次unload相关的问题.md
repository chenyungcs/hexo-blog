title: 记一次unload相关的问题
author: 丘山
tags:
  - JavaScript
  - Event
categories:
  - 前端
date: 2019-02-01 11:27:00
---
> 一如既往，开发中遇到的问题。

需求：
大佬说，我要在用户离开时记录一下。做一个百度的「实时访客」功能
SO：
我们是单页应用
访问每个页面时，发送一个请求，告诉服务器，访问啥页面
如果第一次访问，后端返回一个`ID`作为访问行为的标志
`onunload事件`中发个请求 #14 ，告诉服务器，客户离开了 
下班走人~~

好像没这么简单，于是再来一遍

<!-- more -->

**SO**

假设：浏览器可能不会正常关闭
*那就增加一个「异常」状态，如果超过一段时间没关闭那就异常吧，毕竟微信不是我家开的*

假设：我们产品运行在厉害的（万恶的）微信浏览器上，会不会出幺蛾子呢？
*杀千刀的，浏览器内核不同，还好有兼容指南*
![image](https://user-images.githubusercontent.com/10740017/42142651-21865800-7de3-11e8-9b50-035e671ce9c7.png)

假设：刷新呢？也会执行`onunload`
*好办，记录一下id到`storage`，如果刷新肯定会再次发「访问请求」，把这个id传给后端* 

看似一切都好办，其实坑常在。由于操作的特殊性，浏览器的安全策略，会阻止一些操作，例如：

## 在ios环境下，`unload` 中执行 `localStorage`就是不得行

`storage` 有两种：`session` 和 `local`，都是基于`cookie`，两者api都一样，区别就在于：`session`的生命周期是一次会话；而`localStorage`，需要手动清除。对于浏览器来说，`localStorage`会导致内存增加，影响性能，so，我猜，这个用户无感知（意想不到）的操作，ios是不允许的。

于是，尝试`session`，正好，该特性十分适合这个需求。浏览器会话只会在关闭时结束，所以理所当然，离开时，id也就销毁的，而刷新则还在，十分契合。
 
## `unload` 靠谱吗？

> 之前提到过，ios平台下，`unload`只在刷新时执行

那`beforeUnload`呢？

![image](https://user-images.githubusercontent.com/10740017/42143271-db56f5d4-7de6-11e8-8ec9-fed97eb45a13.png)

在MDN上寻觅后，发现`pageHide`与之同效，与`pageHide` 对应的api有`pageShow`

![image](https://user-images.githubusercontent.com/10740017/42143564-46368670-7de8-11e8-8329-36c4329e1c2c.png)


是时候，发挥咱们前端的真实作用了...
```
const eventName = $deviceTypeIs('ios') ? 'pagehide' : 'unload';

// 监听刷新
window.addEventListener(eventName, (e) => {
    /* do */
});
```

## 总结

- 使用 `unload` 和 `pageHide`
- `localStorage` 和 `sessionStorage`
