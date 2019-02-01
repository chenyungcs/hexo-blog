title: 如何处理IOS端输入法导致fixed元素显示异常？
author: 丘山
date: 2019-02-01 10:52:19
tags:
---
手机端网页一个比较常见的交互场景就是输入，输入由`输入框`和`输入法弹窗UI`共同完成，为了更好的交互，手机端浏览器一般会自动将`输入框`滚动到屏幕中央，使用户能看到输入内容。但不同浏览器处理方式并不一样，导致UI多端不统一

![image](https://user-images.githubusercontent.com/10740017/50135310-aff37f80-02ce-11e9-8fe3-8429ba97f1b7.png)

<!-- more -->

> 描述：点击输入框框，弹出输入法框，按钮可能会被遮住。**而在大部分安卓机型中，按钮是正常显示（图三）的**

差异分析：ios

通过usb连接mac和ios，可以实现页面调试，通过调试

```
document.body.getBoundingClientRect()
// _> DOMRect {x: 0, y: -298, width: 320, height: 434, top: -298, …}
```

可以看出`body`整体上移了200多个像素，但**高度却还是屏幕可视高度**。

找到原因了，那么就可以出解决方案了。有什么web api 可以解决吗？有没有较好的解决方案呢？

web api我没有找到，本身这是个别浏览器内核不同实现，我们的业务中，按钮不需要显示在输入框上面，所以问题相对简单了。

```
// 光标聚焦，文本框回归文档流
handleFocus () {
    this.$refs.footer.style.position = 'static';
}

// 光标丢失，文本框脱离文档流
handleBlur () {
    this.$refs.footer.style.position = 'fixed';
}
```

这样处理，还能保证，各浏览器显示效果一致。

ps:这样会损失一点点小小的性能（回流）


## 🤔如果按钮一定要显示到底部？

> 由于业务不需要实现，没去实践，只是简单分析下实现

### 通过js控制按钮位置

- 判断平台
- 光标聚焦后，获取`document.body.getBoundingClientRect`用于计算
- 设置按钮距离顶部距离

由于键盘弹出有过渡时间，所以这个方法体验可能不是太好

### 将按钮和固定区域（滚动区域）分为两块

```
<div class="wrapper">
    <!-- 高度根据wrapper变 -->
    <div class="content"></div>
    <!-- 固定高度，位于底部 -->
    <button class="btn-bottom"></button>
</div>
```

## 🤔键盘隐藏后，页面下方留白

> ios 下，键盘隐藏后，会在页面下方留下（撑开）一片空包区域，滑动屏幕后便会消失，很奇怪。没仔细研究，感觉应该也是浏览器实现问题吧？想到一个hack的解决办法`模拟滑动屏幕的动作`

```
handleBlur () {
    document.body.scrollBy(0, -1);
},
```