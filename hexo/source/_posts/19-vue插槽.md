---
title: vue 插槽
date: 2023-12-26 11:18:09
tags:
    - vue
categories:
    - vue
      - 文档
---

## 基础

插槽内容可以为文本、模版、组件

``` html
<!-- 父组件 -->
<FancyButton>
  Click me! <!-- 插槽内容 -->
</FancyButton>
```

``` html
<!-- 子组件 -->
<button class="fancy-btn">
  <slot></slot> <!-- 插槽出口 -->
</button>
```

渲染结果：

``` html
<button class="fancy-btn">Click me!</button>
```

## 默认内容

子组件的 ```<slot></slot>``` 标签之间是默认值

``` html
<button type="submit">
  <slot>Submit</slot> <!-- 默认内容 -->
</button>
```

## 具名插槽

当一个组件包含多个插槽出口时，需要使用 name 来给各个插槽分配唯一的 ID，没有提供 name 的插槽会隐式地命名为“default”。

``` html
<!-- 子组件 -->
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot> <!-- name 为 default -->
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

在父组件中，使用 v-slot 或者 # 加上插槽名，来指定插槽内容。

``` html
<!-- 父组件 -->
<BaseLayout>
  <template v-slot:header>
    <!-- header 插槽的内容 -->
  </template>
  <template #footer>
    <!-- footer 插槽的内容 -->
  </template>
</BaseLayout>
```

## 动态插槽名

可定义动态插槽名：

``` html
<base-layout>
  <template v-slot:[dynamicSlotName]>
  </template>

  <!-- 缩写 -->
  <template #[dynamicSlotName]>
    ...
  </template>
</base-layout>
```

## 作用域插槽

可以向插槽出口传递 attributes

### 默认插槽

``` html
<!-- 子组件 -->
<div>
    <slot :text="message" :count="1"></slot>
</div>
```

``` html
<!-- 父组件 -->
<MyComponent v-slot="slotProps">
    {{ slotProps.text }} {{ slotProps.count }}
</MyComponent>

<!-- 或者 -->
<MyComponent v-slot="{ text, count }">
    {{ text }} {{ count }}
</MyComponent>
```

### 具名插槽

``` html
<!-- 父组件 -->
<MyComponent>
  <template #header="headerProps">
    {{ headerProps }}
  </template>

  <!-- 不缩写 -->
  <template v-slot:default="defaultProps">
    {{ defaultProps }}
  </template>

  <template #footer="footerProps">
    {{ footerProps }}
  </template>
</MyComponent>
```

``` html
<!-- 子组件 -->
<slot name="header" :msg1="hello1"></slot>
<slot :msg2="hello2"></slot> <!-- name 为 default -->
<slot name="header" :msg3="hello3"></slot>
```
