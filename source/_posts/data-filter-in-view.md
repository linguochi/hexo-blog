---
title: View 2.0中的数据过滤器
author: 林国池
abbrlink: 7df4e520
date: 2020-03-10 10:01:16
tags:
---
# 数据过滤器

OneNET View 2.0 支持用户自定义过滤器，用户可以自行写入`javascript`代码，借助过滤器实现数据结构转换、筛选和一些简单的计算。

> 当前暂不支持es6及更高版本的ECMAScript语法

当出现以下情况，您可能需要使用数据过滤器：

1. 数据源的值与组件中格式不相同；

2. 一个组件需要同时使用多个数据源；

点击数据过滤器，勾选复选框，启用数据过滤器功能。在代码编辑区填写您的代码。过滤器在使用之前会先进行代码安全监测，无法通过安全监测的过滤器将不能使用。

## 数据过滤器类型

OneNET View 2.0 包含两种类型的过滤器：

`私有过滤器`：每个组件各自拥有一个不影响其他组件的过滤器，且这个过滤器只能这个组件自己使用。

`全局过滤器`：可以设计一些每个组件都可选择是否使用的过滤器。这些过滤器一旦被修改，将影响到所有使用到它的组件。

## 过滤器传参

每个 OneNET View 2.0 过滤器都是一个`javascript`函数，我们预先定义了这个过滤器函数的两个参数`data`和`rootData`:

`data`:当前组件选中的数据源数据

`rootData`:包含所有数据源数据的根对象

`rootData` 是 `OneNET View 2.0` 的数据源列表，它是一个以 `数据源Id`-`数据值列表` 为键值对 的对象，包含了该项目中`所有的`数据源，其中`OneNET`、`OneNET1`、`staticDemo`是`数据源 Id`，紧随其后的是该数据源的当前数据。

```json
{
  "OneNET": [
    {
      "update_at": "2019-02-28 14:10:47",
      "id": "line2",
      "create_time": "2017-12-08 15:08:40",
      "current_value": 65,
      "at": "2019-02-28 14:10:47",
      "value": 65
    }
  ],
  "OneNET1": [
    {
      "create_time": "2017-11-22 14:21:15",
      "id": "map"
    }
  ],
  "staticDemo": [
    {
      "x": "2010/01/01 00:00:00",
      "y": 375,
      "s": "1"
    },
    {
      "x": "2010/01/01 00:00:00",
      "y": 180,
      "s": "2"
    },
    ...
  ]
}
```

以这个 `OneNET View 2.0数据源列表`为例。假设通过数据源选择下拉框选中的是`OneNET`,则 参数`data` 的值是

```js
[
  {
    update_at: '2019-02-28 14:10:47',
    id: 'line2',
    create_time: '2017-12-08 15:08:40',
    current_value: 65,
    at: '2019-02-28 14:10:47',
    value: 65,
  },
];
```

## 过滤器示例
### 一个组件里同时采用两个以上数据源
仍以上文提到的数据源为例，假设一个组件需要数据源`OneNET`和数据源`staticDemo`的数据，则我们将数据源选择下拉框选中`OneNET`，然后在过滤器中这么写：
```javascript
// 代码中 data 取到的是我们数据源下拉框选中的OneNET数据源
var staticDemo = rootData.staticDemo; // 同理其他数据源如OneNET1 可以通过 rootData.OneNET1 取到
function last(arr) {
  var len = arr ? arr.length : 0;
  if (len) return arr[len - 1];
}

return [{
    "value": last(staticDemo).x + ' ' + last(data).value
}]
```
