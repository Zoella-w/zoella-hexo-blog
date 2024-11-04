---
title: vue3 组件 v-model
date: 2023-12-15 20:27:47
tags:
    - vue3
    - vue
categories:
    - vue3
      - 文档
---

## 基础

父组件：

``` html
<CustomInput v-model="searchText" />

<!-- 被展开为： -->
<!-- <CustomInput
    :model-value="searchText"
    @update:model-value="newValue => searchText = newValue" /> -->
```

子组件：

在引用的子组件中使用 v-model 指令时，子组件的 modelValue prop 默认用于传递输入值，并且会触发名为 update:modelValue 的事件来更新该属性。

``` html

<script setup>
// 声明属性 modelValue，这是父组件通过 v-model 传给当前组件的 prop
defineProps(['modelValue']);
// 声明事件 update:modelValue，用于在当前组件中触发父组件的更新操作
defineEmits(['update:modelValue']);
</script>

<template>
  <!-- 将 modelValue 作为其值，并在输入时触发 update:modelValue 事件 -->
  <input
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>
```

## 如何更改默认名 modelValue

父组件：

``` html
<MyComponent v-model:title="bookTitle" />
```

子组件：

``` html
<!-- MyComponent.vue -->
<script setup>
defineProps(['title']);
defineEmits(['update:title']);
</script>

<template>
  <input
    type="text"
    :value="title"
    @input="$emit('update:title', $event.target.value)"
  />
</template>
```

## 自定义 v-model 修饰符

创建一个自定义修饰符 capitalize，自动将 v-model 绑定输入字符串的第一个字母转为大写：

``` html
<MyComponent v-model.capitalize="myText" />
```

``` html
<script setup>
// modelModifiers prop 包含了 capitalize 且其值为 true
// 因为它在模板中的 v-model 绑定 v-model.capitalize="myText" 上被使用了
const props = defineProps({
    modelValue: String,
    modelModifiers: {
        default: () => ({}) // 默认值为空对象
    },
});

defineEmits(['update:modelValue']);

// 将首字母大写
function emitValue(e) {
    let value = e.target.value;
    if(props.modelModifiers.captialize) {
        value = value.charAt(0).toUpperCase() + value.slice(1);
    }
    emit('update:modelValue', value);
}
</script>

<template>
    <!-- 每次 <input /> 元素触发 input 事件时触发 emitValue -->
    <input
        type="text"
        :value="modelValue"
        @input="emitValue"
    />
</template>
```

### 带多个不同参数的 v-model 修饰符

``` html
<UserName
    v-model:first-name.capitalize="first"
    v-model:last-name.uppercase="last"
/>
```

``` html
<script setup>
const props = defineProps({
    firstName: String,
    lastName: String,
    firstNameModifiers: {  // firstName + Modifiers
        default: () => ({}) 
    },
    lastNameModifiers: {  // lastName + Modifiers
        default: () => ({})
    },
});

defineEmits(['update:firstName', 'update:lastName']);

console.log(props.firstNameModifiers); // { calitalize: true }
console.log(props.lastNameModifiers); // { uppercase: true }
</script>
```
