---
title: 4-vue3计算属性
date: 2023-11-07 18:46:36
tags:
    - vue3
    - vue
categories:
    - vue3
      - 文档
---

## 基础

computed() 方法接受一个 getter 函数，返回一个计算属性 ref

因为 ref 会在模板中自动解包，所以在表达式中引用无需 .value

``` html
<script setup>
import { reactive, computed } from 'vue';

const author = reactive({
    name: "Zoella",
    books: ['vue2', 'vue3', 'vue4']
});

const hasBookPublished = computed(() => {
    return author.books.length > 0 ? 'yes' : 'no';
});
</script>

<template>
  <div>
    <span>{{ author.name }}</span> has published books: 
    <span>{{ hasBookPublished }}</span>
  </div>
</template>
```

## 计算属性缓存 VS 方法

``` html
<p>{{ calculateBooksMessage() }}</p>
```

``` js
function calculateBooksMessage() {
  return author.books.length > 0 ? 'yes' : 'no';
}
```

计算属性比方法节省性能。

将与上述相同的函数定义为方法，结果和计算属性相同，然而 **计算属性值会基于其响应式依赖被缓存**，只要```author.books```不变，就不会重复执行 getter 函数。但是方法总会在重渲染发生时再次执行函数。

## 可写计算属性

计算属性默认为只读。特殊场景下会用到“可写”的计算属性。

``` html
<script>
import { ref, computed } from 'vue';

const firstName = ref('John');
const lastName = ref('Doe');

const fullName = computed({
  // getter
  get() {
    return firstName.value + ' ' + lastName.value;
  },
  // setter
  set(newValue) {
    [firstName.value, lastName.value] = newValue.split('');
  }
})
</script>
```

当运行 ```fullName.value = 'Zoella Wang'``` 时，setter会被调用，firstName 和 lastName 会随之更新。
