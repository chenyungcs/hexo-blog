title: 调试接口很痛苦有木有，小试一下Mock
author: 丘山
tags:
  - 工具
  - JavaScript
categories:
  - 前端
date: 2019-02-01 14:58:00
---
> 开发中，常遇到A要接口、B要接口、C要接口。环境却只有一个，一旦切换，都将导致BC接口访问失败啥的，下面，我准备把Mock 用起来。

## 安装

[https://github.com/nuysoft/Mock/wiki/Getting-Started](https://github.com/nuysoft/Mock/wiki/Getting-Started)

根据自己项目调整就好了。

html
```
<script src="http://mockjs.com/dist/mock.js"></script>
<!-- 主要是进行配置：timeout 目前唯一的配置项 -->
<script src="./js/mock/index.js"></script>
<!-- 分接口的Mock规则 -->
<script src="./js/mock/MallStatistics.js"></script>
```

`MallStatistics.js`

```
/**
 * 请求成功的返回格式
 */
function successBody (body, limit = 0) {
    return {
        [`data${limit ? '|' + limit : ''}`]: body,
        'code': 0
    };
}

/**
 * 请求失败的返回格式
 */
function errorBody (body, code = 1001) {
    return {
        'data': body,
        'code': code
    };
}

var liveRule = new RegExp('/mock/mall/live?.+', '');

/**
 * 实时访客
 */
Mock.mock(liveRule, 'get', function ({ body }) {
    var { limit = 20, offset, date } = JSON.parse(body);

    // 模拟数据
    var data = Mock.mock(successBody(
        {
            [`meta|${limit}`]: [{
                'access_time': `${date} @TIME`,                     // 访问时间
                'client_number': '@INTEGER(10000, 99999)',      // 客户编号
                'client_name': '@CWORD(6,20)',                  // 客户名称（店名）
                'main_account': '@CNAME',                       // 主账号
                'phone': '182@INTEGER(1000, 9999)@INTEGER(1000, 9999)', // 手机
                'dc_name|1': ['楼兰', '大燕京', '你看', '数据啊', ''], // 营业部
                'place_order|1': [true, false],                 // 是否下单
                'residence_time': '@INTEGER(5, 10000)',         // 停留时间？这个是否从access_url中统计（前端处理）
                'is_active|1': [true, false],                   // 是否正在访问
                'access_url|1-50': [{                          // 访问记录
                    'access_time': '@TIME', // - 访问时间
                    'residence_time': '@INTEGER(5, 10000)',     // - 停留时间
                    'url': 'http://mall.balabala.cn/#/@NAME'       // - url

                }]
            }],
            'total': `@INTEGER(${limit}, ${limit*10})`
        }
    ));
    data.data.meta = _.sortBy(data.data.meta, 'access_time').reverse();

    return data;
});
```

## 需要注意的语法规范

*数据模板中的每个属性由 3 部分构成：属性名、生成规则、属性值*

```
// 属性名   name
// 生成规则 rule
// 属性值   value
'name|rule': value
```

#### 生成规则`|`

当`|`规则为空时，表示为`|0`

#### 占位符

`Mock`内置了强大的`Random`对象。可通过对象调用，也可以通过「占位符」，例如：

```
// 生成一个随机英文名
{
    'name': Mock. Random.name()
}

// 那这样呢？
{
    'data|10': [{
        'name': Mock.Random.name()
    }]
}
```

可见连续生成了10个同样的数据，我们需要随机生成不同的。有两种方法:

```
// 通过function
{
    'data|10': [{
        'name': function () {
            return Mock.Random.name();
        }
    }]
}

// 通过占位符
{
    'data|10': [{
        'cname': '@CNAME',
        'englishName': '@FIRST@MIDDLE@LAST'
    }]
}
```

可见，通过占位符调用`Random`里的方法，就是`@全大写的方法名`
对了，你还可以`@已有规则`，例如：

```
{
    'data|10': [{
        'myName': '@CNAME',
        'store': '@myName的小店'
    }]
}
```

## `mock(rurl?, rtype?, template|function( options ))`方法的使用

```
rurl      -> url规则
rtype   -> 请求类型
```

方法很简单，很好使用，只有两点需要注意的。

1. `get`请求下，参数也会被视作`url`的一部分，例如`user?id=1`，这也是开头我使用正则的原因。 
👇下面的例子将无法正确匹配

```
// mock
/user

// http
$.ajax.get('user/', {
    id: 1,
    name: '李四'
});
```

2. 请求类型在大多数情况下是不需要的。
但，如果你的接口是`RESTful风格`，加上Type就十分必要的

```
DELETE : /user
GET:          /user
POST:       /user
```

## `timeout` 重要性

一般，我们的页面一加载，就会请求一波后端接口，受网络影响，会延迟那么一会儿。现在我们使用的`Mock`会在被调用的时候，马上去生成数据，由于生成数据算法影响 **会卡一波**。所以，别怪我没提醒你，乖乖的去设置你的`timeout`。

```
Mock.setup({
    'timeout': 500
});
```

**对了，那就顺便记录一下：**
当生成复杂列表时：超过100条会明显卡顿。

## 约定好`Model`

> 开发前，一定和后端大佬约定好`Model`，接口地址、参数。

大概就是酱紫😯，总结下

- 注意语法规则
- 注意url规则和请求类型
- 设置`timeout`
- 约定`Model`


