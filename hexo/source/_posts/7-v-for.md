---
title: v-for
date: 2023-11-29 10:41:57
tags: 
    - vue
categories:
    - vue
      - 文档
---

## 基础

``` html
<li v-for="(item, index) in items"></li>
```

可以使用 of 代替 in：

``` html
<li v-for="(item, index) of items"></li>
```

支持解构语法：

``` html
<li v-for="{message} in items">
    {{ message }}
</li>

<li v-for="({message}, index) in items">
    {{ message }}
</li>
```

## 遍历对象属性

使用 v-for 遍历对象属性，遍历的顺序和 Object.keys() 返回值的顺序一致

三个参数依次为 索引、属性名、属性值：

``` js
const myObject = reactive({
  title: 'How to do lists in Vue',
  author: 'Jane Doe',
  publishedAt: '2016-04-10'
});
```

``` html
<li v-for="(index, key, value) in myObject">
{{ index }}. {{ key }}: {{ value }}
</li>
```

结果为：

``` txt
0. title: How to do lists in Vue
1. author: Jane Doe
2. publishedAt: 2016-04-10
```

## v-for 与 v-if

v-if 比 v-for 优先级高

错误用法：

``` html
<!-- 此时 v-if 的 todo 还没有定义 -->
<li v-for="todo in todos" v-if="!todo.isComplete">
{{ todo.name }}
</li>
```

正确用法：

``` html
<template v-for="todo in todos">
   <li v-if="!todo.isComplete">
   {{ todo.name }}
   </li>
</template>
```

## 通过 key 管理状态

Vue 默认按照 “就地更新” 的策略更新通过 v-for 渲染的列表。当数据项的顺序改变时，Vue 不会随之移动 DOM 元素的顺序，而是就地更新每个元素，确保它们在原本指定的索引位置上渲染。

默认模式是高效的，但只适用于 **列表渲染输出的结果不依赖子组件状态或者临时 DOM 状态 (例如表单输入值) 的情况**。

推荐在任何时候为 v-for 提供一个 key attribute，除非所迭代的 DOM 内容非常简单。
