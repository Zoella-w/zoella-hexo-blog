---
title: 10-v-model
date: 2023-12-05 10:31:56
tags: 
    - vue
categories:
    - vue
---

## 基础

``` html
<input
    :value="text"
    @input="event => text = event.target.value" />
```

用 v-model 简化：

``` html
<input v-model="text" />
```

## 类型

### 单行

``` html
<p>{{ message }}</p>
<input v-model="message" placeholder="edit" />
```

### 多行

``` html
<p>{{ message }}</p>
<textarea v-model="message" placeholder="edit"></textarea>
```

### 复选框

label 标签 for 属性的作用：用户点击 label 标签时，浏览器会将焦点转移到与 for 属性值相匹配的表单控件上，从而提高表单的可访问性和易用性。

``` html
<div>Checked names: {{ checkedNames }}</div>

<input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
<label for="jack">Jack</label>

<input type="checkbox" id="john" value="John" v-model="checkedNames">
<label for="john">John</label>

<input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
<label for="mike">Mike</label>
```

### 单选

``` html
<div>Picked: {{ picked }}</div>

<input type="radio" id="one" value="One" v-model="picked" />
<label for="one">One</label>

<input type="radio" id="two" value="Two" v-model="picked" />
<label for="two">Two</label>
```

### 选择器

单个：

``` html
<div>Selected: {{ selected }}</div>

<select v-model="selected">
  <option disabled value="">Please select one</option>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
```

多个（将值绑定到数组上）：

``` html
<div>Selected: {{ selected }}</div>

<select v-model="selected" multiple>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
```

## 修饰符

### .lazy

.lazy 修饰符表示在 "change" 事件而不是 "input" 事件触发时更新：

这意味着数据将在失去焦点后才会同步到视图中，而不是每次输入时都同步。可以用于减少输入框频繁更新视图的情况，尤其是在处理大量输入时可以提高性能。

``` html
<input v-model.lazy="msg" />
```

### .number

让用户输入自动转换为数字，如果该值无法被 parseFloat() 处理，则将返回原始值。

number 修饰符会在输入框有 type="number" 时自动启用。

``` html
<input v-model.lazy="msg" />
```

### .trim

默认自动去除用户输入内容中两端的空格：

``` html
<input v-model.trim="msg" />
```
