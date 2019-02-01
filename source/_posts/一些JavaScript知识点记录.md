title: 一些JavaScript知识点记录
author: 丘山
tags:
  - JavaScript
categories:
  - 前端
date: 2019-02-01 15:05:00
---
- 闭包
- [谈谈数据类型](#simpleType)
- [垃圾回收机制](#trash)
- event loop
- [数组](#array)
- [正则](#regular)

## 喜闻乐见的闭包

> 函数作用域可以通过作用域链相互关联起来，函数体内部的变量都可以保存在函数作用域内，这种特性称为闭包！

通常的套路是，面试官会给出一段代码

```
for (var i = 0; i < 5; i++) {
    setTimeout(() => console.log(i), 0);
}

// 输出？
// 5
// 5
// 5
// 5
// 5
```

面试官：非常好，为什么？
小白我：因为没有闭包。
面试官（露出慈祥的笑容）：好，将上面代码改成闭包并且解释下闭包。
balabala...

```
// 小白我
for (var i = 0; i < 5; i++) {
    (() =>setTimeout(() => console.log(i), 0))();
}

// 正确解法
// `event loop` 执行顺序，同步代码 > micro task > macro task，所以将`setTimeout`放入闭包内
// `for循环`内自执行函数（闭包）声明变量保存i值
// 输出`index`变量
for (var i = 0; i < 5; i++) {
    (() => {
	var index = i;
	setTimeout(() => console.log(index), 0);
    })();
}

// es6 解法
for (let i = 0; i < 5; i++) {
    setTimeout(() => console.log(i), 0);
}
```

### `let` 特性
- 不存在变量提升
- 暂时性死区
- 同一作用域不允许重复声明

### 为什么使用闭包？何时使用？

- 保护作用域变量
- 防止污染全局变量

**缺点：**

- 容易[内存泄漏](#trash)
- 更占用内存

**何时用？**

相信如果项目没有使用es6，打开公司的前端源代码，会看到大量如下代码
```
(function () {
    /* todo */
})();
```
上面是一个自执行函数，也是我最早意识到闭包！
所以闭包在需要保护作用域变量、实现私有变量时使用！

关于防止污染全局变量，可以用对象解决，即在全局作用域下声明变量

```
window.GLOBAL = {
    a: {},
    b: {}
};
```

### 什么是作用域链？

![作用域链示意图](https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=785176101,763123448&fm=27&gp=0.jpg)

> JavaScript是基于词法作用域的语言：通过阅读包含变量定义在内的数行源码就能知道变量作用域！具体呈现就像树状结构。

**总结**

这道题其实变相考察应试者的经验！面试官给分：[event loop]() > [异步]() > 闭包。当然无论你回答以上三个答案，最终面试官都会继续往下挖，直到他满意或者你答不上！


## <a name="simpleType">谈谈数据类型</a>

> 包括原始类型、对象类型

**原始类型**
- 数字
- 字符串
- 布尔

**原始值**
- null
- undefined

**对象类型**
- 除原始类型和原始值之外的类型

常见的有 Array, Object, Function, Date ... 

## <a name="trash">JavaScript垃圾回收机制</a>

> 当不再有任何引用指向一个对象，「解释器」就会知道这个对象没用了，然后自动回收它所占用的「内存资源」

### 内存泄漏

> 循环引用时，内存资源得不到释放，就会导致内存泄漏！

```
// 内存泄漏
window.onload = function (){
    var element = document.getElementById("id");
    element.onclick = function (){
        alert(element.id);
    }
}

// 解决办法
window.onload = function (){
    var element = document.getElementById("id");
    element.onclick = function (e){
        alert(e.target.id);
    };
}
```

## <a name="regular">正则</a>

## <a name="array">数组</a>

### 数组创建

数组创建有两种方法

- 字面量
- new Array

`new Array()` 构造函数，接受多个参数。

```
// 字面量
let arrayDemo = [1, 2, 3];

// new
let arrayDemo = new Array(1, 2, 3);
// 值得注意的是，只传一个参数且为「数字」时，会被认定为初始化「一个数字长度」的数组
let arrayDemo = new Array(2); // => [empty x 2]
```

### 方法

#### 会改变自身的方法

> 原始（基本）数组类型是不能改变自身的！比如我们常用的`string`，`charAt` `substr` 这类方法最终也只是返回一个值！同样属于原值类型的还有`number` `boolean`

- push

向数组末尾追加一个或者多个内容，并返回追加后数组长度

```
arrayDemo.push(var1, var2);
```

通常，我们添加一组数据，这组数据很可能是动态的，那能这样吗？

```
arrayDemo.push([1, 2]); // => [[1,2]]
```

有两种解决方案：

```
// es6: 利用数组解构
arrayDemo.push(...[1, 2])

// 利用`apply`特性
[].push.apply(arrayDemo, [1, 2])
```

使用`apply`时，应当注意该函数参数个数是「有限制」的。

- pop 

push 的反向方法，删除数组最后一个内容，并返回删除的内容

- unshift

在数组前面追加一个或者多个内容，并返回数组长度

- shift

删除第一个内容，并返回删除的内容

- splice(start, deleteCount, item1, item2, ...)

该方法通过删除现有元素和/或添加新元素来更改一个数组的内容。并且返回删除的内容。

在`start`到`start + deleteCount`之间添加新内容，直到`item*n`都插入到数组

```
// arrayDemo => [1, 2, 3]
arrayDemo.splice(1, 1, 'n', 'n', 'n');  // =>  [1, "n", "n", "n", 3]

```

- sort(fn)

对数组排序，`fn`执行`length - 1`次，并返回排序后的数组

### 正则匹配返回的数组

属性/元素 | 说明 | 示例
-- | -- | --
input | 只读属性，原始字符串 | cdbBdbsbz
index | 只读属性，匹配到的子串在原始字符串中的索引 | 1
[0] | 只读元素，本次匹配到的子串 | dbBd
[1], ...[n] | 只读元素，正则表达式中所指定的分组所匹配到的子串，其数量由正则中的分组数量决定，无最大上限 | [1]: bB[2]: d

摘自[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)


