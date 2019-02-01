title: '记keyup一个细节 '
author: 丘山
tags:
  - JavaScript
  - Event
categories:
  - 前端
date: 2019-02-01 14:56:00
---
> 需求：在客户端监听keyup事件，实现数据切换

```
// 识别按键，分别执行数据切换、数据提交操作
window.addEventListener('keyup', this.handleKeyUpEvent); 
```

在keyup事件中，会去同步数据同时刷新`dom节点`。数据刷新不及时、页面感觉卡顿，why?
分析按键事件，`keydown -> keyup`，这是两兄弟，对吧？所以keyup肯定是发生在keydown之后啦，所以页面卡顿... emum，具体延迟如下：

```
window.addEventListener('keydown', () => {
	console.time('keyup');
}); 

window.addEventListener('keyup', () => {
	console.timeEnd('keyup');
});

// _> keyup: 156.1640625ms
// _> keyup: 156.760009765625ms
// _> keyup: 47.076904296875ms
```

手速是响应的关键，并且，我觉得keyup事件，没这么简单，会不会还有「释放资源」这种骚操作？**所以keyup处理的事情越简单，越好吧**


```
// 改为keydown，keydown执行多次，节流？还是另行其法
window.addEventListener('keydown', this.handleKeyDown); 
```

keydown 会执行多次，在我的业务场景中，希望点一下按键，执行一下操作，所以，我希望，keydown执行一次，keyup一次，代码如下：

```
let keyUp = true;
window.addEventListener('keydown', () => {
	if (keyUp) {
		keyUp = false;
		// things to do
	}
}); 

window.addEventListener('keyup', () => {
	keyUp = true;
});

```

**最后：** 记得销毁事件

