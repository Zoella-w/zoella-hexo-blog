---
title: 3-vue3响应式
date: 2023-11-06 16:41:05
tags:
    - vue3
    - vue
categories:
    - vue3
      - 文档
---

## ref()

ref() 返回一个包含属性 value 的对象

``` js
import { ref } from 'vue';

export default {
    setup() {
        const count = ref(0);
        function increment() {
            count.value++;
        }
        return {
            count,
            increment
        }
    }
}
```

``` html
<button @click="increment">{{ count }}</button>
```

### 原理

ref 的 .value 属性使得 Vue 可以检测其何时被访问或修改

当一个组件首次渲染时，Vue 会**追踪**在渲染过程中使用的每一个 ref；当一个 ref 被修改时，它会**触发** 追踪它的组件 的重新渲染

``` js
// 伪代码
const myRef = {
    _value: 0,
    get value() {
        track(); // 追踪渲染过程中使用的每一个 ref
        return this._value;
    },
    set value(newValue) {
        this._value = newValue;
        trigger(); // 触发追踪它的组件的重新渲染
    }
}
```

### ``<script setup>``

使用 ``<script setup>`` 简化代码

``` html
<script setup>
import { ref } from 'vue';

const count = ref(0);

function increment() {
    count.value++;
}
</script>

<template>
    <button @click="increment">{{ count }}</button>
</template>
```

## DOM 更新时机

DOM 更新后，立即执行回调函数

``` js
import { nextTick } from 'vue';

async function increment() {
    count.value++;
    await nextTick();
    callback();
}
```

或者

``` js
import { nextTick } from 'vue';

function increment() {
    count.value++;
    nextTick(() => callback());
}
```

## reactive()

reactive() 使对象本身具有响应性，当 ref 的值是一个对象时，会在内层调用 reactive

### 原理

reactive() 返回原始对象的 proxy，允许 Vue 拦截和定义基本操作的行为（如属性查找、赋值、删除等）

reactive() 的返回值和原始对象不相等

``` js
const raw = {};
const proxy = reactive(raw);
console.log(raw === proxy); // false
```

### reactive() 的局限性

（1）有限的值类型：只能用于对象类型（对象、数组、Map、Set），不能用于原始类型；

（2）不能替换整个对象：替换整个对象会导致响应式连接丢失；

（3）对解构操作不友好：解构后的变量会丢失响应式连接。

## ref 解包

### ref 作为 reactive 对象属性

ref 作为响应式对象的属性时，就像一个普通的属性

``` js
const count = ref(0);
const state = reactive({ count });
console.log(state.count); // 0

state.count = 1;
console.log(state.count); // 1
```

### 在模板中解包

在模板渲染上下文中，只有顶级的 ref 才会被解包

第二个不符合预期是因为，obj.id 未被解包，仍是一个 ref 对象

``` js
const count = ref(0);
const obj = {
    id: ref(1)
};
```

``` html
<!-- 符合预期 —— 2 -->
{{ count + 1 }}
<!-- 不符合预期 —— [object Object]1 -->
{{ obj.id + 1 }}
```

为了解决该问题，需要将 id 结构为顶级属性

``` js
const { id } = obj;
```

``` html
<!-- 符合预期 —— 2 -->
{{ id + 1 }}
```
