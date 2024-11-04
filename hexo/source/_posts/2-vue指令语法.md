---
title: vue 指令语法
date: 2023-11-06 16:04:06
tags:
    - vue
categories:
    - vue
      - 文档
---

## 指令

种类：v-html, v-bind, v-on, v-model, v-slot ...

指令语法如下图所示：
![指令语法](https://img20.360buyimg.com/img/jfs/t1/89597/35/41403/16292/65489fc5Fb2d8b302/d64ca449c9e1d57b.png)

### 动态参数

（1）动态参数的值应为字符串或 null（null 会移除该绑定）

（2）动态参数的名称避免使用大写字母，因为会被强制转为小写（someAttr -> someattr）

``` html
<a :[attributeName]="url"> ... </a>

<a @[eventName]="doSomething">
```

### 修饰符

修饰符为以点开头的特殊后缀，比如：.prevent 修饰符会告知 v-on 指令对触发的事件调用 event.preventDefault()

``` html
<div @submit.prevent="onSubmit"></div>
```

## v-html

span 的内容会被替换为 rawHtml 属性的值，其中的数据绑定会被忽略（注意：使用这种方法容易造成 XSS 漏洞）

``` html
<span v-html="rawHtml"></span>
```

## v-bind

``` html
<div v-bind:id="dynamicId"></div>
<!--简写-->
<div :id="dynamicId"></div>
```

### 动态绑定多个值

``` js
const objOfAttrs = {
    id: 'container',
    class: 'wrapper'
};
```

``` html
<div :objOfAttrs></div>
```

比如：动态 style class

``` html
<div :style="{class1: isClass1, class2: isClass2}"></div>
```

## v-on

``` html
<div v-on:click="onClick"></div>
<!--简写-->
<div @click="onClick"></div>
```

## v-if

v-if, v-else-if, v-else

注意：v-if 的优先级大于 v-for，二者不建议同时使用（详见 v-for）

### template 上的 v-if

如果想要切换不止一个元素，可以在元素外包一个 ``<template>``，并用 v-if 控制（v-show 不能在 template 上使用）

因为 ``<template>`` 是一个不可见的包装器元素，渲染的结果中不会包含该元素

### v-if & v-show

（1）原理：v-if 切换时，条件区块会被销毁与重建；v-show 切换时，只会切换 display 属性

（2）惰性：v-if 是惰性的，如果初始为 false 则不渲染；v-show 初始始终会渲染

（3）场景：v-if 初始渲染开销较好，不会频繁切换时使用；v-show 切换开销较小，频繁切换时使用
