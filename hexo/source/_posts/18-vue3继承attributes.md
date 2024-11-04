---
title: vue3 继承 attributes
date: 2023-12-21 20:33:12
tags:
    - vue3
    - vue
categories:
    - vue3
      - 文档
---

## 禁用 attributes 继承

如果不想一个组件自动继承 attribute，可在组件选项中设置 ```inheritAttrs: false```

``` html
<script setup>
defineOptions({
    inheritAttrs: false
});
</script>
```

透传进来的 attribute 可在模版的表达式中用 $attrs 访问：

``` html
<span>attribute: {{ $attrs }}</span>
```

### 示例

常见的需要禁用 attribute 继承的场景是：attribute 需要应用在根节点以外的其他元素。

比如：希望透传的 attribute 应用在内层的节点而非外层。

``` html
<div class='btn-wrapper'>
    <button class='btn' v-bind="$attrs">click me</button>
</div>

<script setup>
defineOptions({
    inheritAttrs: false
});
</script>
```

## 多根节点的 attributes 继承

需要显示绑定 $attrs，否则会有警告

``` html
<header>...</header>
<main v-bind="$attrs">...</main>
<footer>...</footer>
```

## 在 JS 中访问透传的 attributes

使用 useAttrs() API 访问组件的所有透传 attribute

``` html
<script setup>
import { useAttrs } from 'vue';

const attrs = useAttrs();
</script>
```

但是这里的 attrs 对象不具有响应式，如有需要：

（1）可使用 prop；

（2）可使用 onUpdated() 在每次更新时获取最新的 attrs。
