---
title: 为什么 vue 组件添加 scoped 后，某些组件样式不生效
date: 2023-11-14 17:46:52
tags:
    - vue
    - 实践问题
categories:
    - vue
      - 样式
---

## 简述

在父组件中修改子组件的某些样式，发现不生效，删去``<style scoped></style>``中的 scoped 后生效。

## 原因

scoped 实现样式隔离的原理为：

编译时，父组件的所有标签、子组件的根标签、以及所有的样式 都会加上特殊的标识；

因为子组件内部的标签都没有此种标识，所以样式就不会生效。

## 实例

### 不添加 scoped

``` html
<!-- 父组件 -->
<template>
  <div class="parent">
    <p>Here is parent component</p>
    <TestScoped />  
  </div>
</template>
<style>
.parent {
  color: deepskyblue;
}
</style>
```

``` html
<!-- 子组件 -->
<template>
  <div class="son">
    <p>Here is son component</p>
  </div>
</template>
```

编译后：

``` html
<div class="parent">
    <p>Here is parent component</p>
    <div class="son">
        <p>Here is son component</p>
    </div>
</div>
```

``` css
p {
  color: deepskyblue;
}
```

### 添加 scoped

编译后：

``` html
<div data-v-7ba5bd90 class="parent">
    <p data-v-7ba5bd90>Here is parent component</p>
    <div data-v-7ba5bd90 class="son">
        <!-- 没有标识，所以不生效 -->
        <p>Here is son component</p>
    </div>
</div>
```

``` css
p[data-v-7ba5bd90] {
    color: deepskyblue;
}
```

## 解决方法

### 深度作用选择器

使用 /deep/ 或者 ::v-deep

``` css
/deep/ p {
  color: deepskyblue;
}
/* 或者 */
::v-deep p {
  color: deepskyblue;
}
```

编译后：

html 结果不变，样式代码变化

``` css
[data-v-7ba5bd90] p {
  color: deepskyblue;
}
```

### 使用无 scoped 的 style

``` html
<style scoped>
p {
  color: deepskyblue;
}
</style>

<style>
/* 生效样式代码 */
</style>
```
