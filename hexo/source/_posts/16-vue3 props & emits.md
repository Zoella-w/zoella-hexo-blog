---
title: vue3 props & emits
date: 2023-12-15 20:26:11
tags:
    - vue3
    - vue
categories:
    - vue3
      - 文档
---

## props

注意：props 为单向数据流，从父组件流子组件，所以不应该在子组件中修改 props。

如有需要：

1. 用另一个属性接收该 prop 的初始值；
2. 基于该 prop 定义一个计算属性；
3. 向父组件抛出一个事件。

### 声明

使用字符串数组：

``` html
<script setup>
const props = defineProps(['foo']);
</script>
```

使用对象：

``` html
<script setup>
const props = defineProps({
    title: String,
    likes: Number
});
</script>
```

### 静态 prop

除了静态字符串，都应该使用变量进行传递。

``` html
<BlogPost title="this is a title" />

<!-- <BlogPost :likes="42" /> -->
<BlogPost :likes="post.likes" />

<!-- <BlogPost is-published="true" /> -->
<BlogPost :is-published="post.isPublished" />

<!-- <BlogPost ids="[1, 2, 3]" /> -->
<BlogPost :ids="post.ids" />

<!-- <BlogPost author="{ name: 'Zoella', age: 22 }" /> -->
<BlogPost :author="post.author" />
```

### 一个对象绑定多个 prop

使用无参数的 v-bind，而不是 :prop-name

``` js
const post = {
    id: 1,
    title: "this is title",
};
```

``` html
<BlogPost v-bind="post" />
```

### prop 校验

``` js
defineProps({
  // 多种可能的类型检查（给出 `null` 和 `undefined` 值则会跳过任何类型检查）
  propA: [String, Number],
  // 必传，且为 String 类型，默认值为 'defualtVal'
  propB: {
    type: String,
    required: true,
    default: 'defaultVal'
  },
  // 对象类型的默认值
  propC: {
    type: Object,
    // 对象或数组的默认值，必须从一个工厂函数返回。
    // 该函数接收组件所接收的原始 prop 作为参数。
    default(rawProps) {
      return { message: 'hello' }
    }
  },
  // 自定义类型校验函数
  propD: {
    validator(value) {
      return ['success', 'warning', 'danger'].includes(value)
    }
  },
  // 函数类型的默认值
  propE: {
    type: Function,
    default() {
      return 'Default function'
    }
  }
});
```

### Boolean 类型转换

``` js
defineProps({
  disabled: Boolean
});
```

使用时：

``` html
<!-- 等同于 <MyComponent :disabled="true"></MyComponent> -->
<MyComponent disabled></MyComponent>

<!-- 等同于 <MyComponent :disabled="false"></MyComponent> -->
<MyComponent></MyComponent>
```

## emit

### 事件校验

``` html
<script setup>
const emit = defineEmits({
  // 无校验
  click: null,

  // 校验 submit 事件
  submit: ({ email, password }) => {
    if (email && password) {
      return true;
    } else {
      console.warn('Invalid submit event payload!')
      return false;
    }
  }
});

function submitForm(email, password) {
  // 抛出 submit 事件，携带 email, password 参数
  emit('submit', { email, password });
}
</script>
```
