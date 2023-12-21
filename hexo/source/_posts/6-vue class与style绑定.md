---
title: 6-vue class与style绑定
date: 2023-11-28 19:03:13
tags:
    - vue3
    - vue
categories:
    - vue
      - 样式
    - vue3
      - 文档
---

## 绑定一个返回对象的计算属性

``` js
const isActive = ref(true);
const error = ref(null);

const classObject = computed(() => ({
    active: isActive.value && !error.value,
    'text-danger': error.value && error.value.type === 'fatal'
}));
```

``` html
<div :class="classObject"></div>
```

## 子组件继承父组件传入的class

### 有一个根元素的组件

子组件的根元素，在渲染时会添加父组件的 class。

### 有多个根元素的组件

子组件中 ```:class='$attrs.class'``` 的根元素，在渲染时会添加父组件的 class。

``` html
<!-- 子组件 MyComponent.vue -->
<p :class="$attrs.class">one root element</p>
<span>another root element</span>
```

``` html
<!-- 父组件 -->
<MyComponent class="fatherClass"/>
```

渲染后结果：

``` html
<p class="fatherClass">one root element</p>
<span>another root element</span>
```

## 內联样式

### 绑定对象

``` js
const styleObject = reactive({
    color: "red",
    fontSize: "13px"
});
```

``` html
<div :style="styleObject"></div>
```

### 绑定对象数组

``` html
<div :style="[baseStyle, overriddingStyle]"></div>
```

``` js
const baseStyle = reactive({
    color: "red",
    ...
});
const overriddingStyle = reactive({
    color: "black",
    ...
});
```
