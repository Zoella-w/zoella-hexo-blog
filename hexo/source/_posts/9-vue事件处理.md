---
title: vue 事件处理
date: 2023-11-30 18:50:13
tags: 
    - vue
categories:
    - vue
      - 文档
---

v-on:click="" 缩写为 @click

## 在內联事件处理器中访问事件参数

在內联事件中访问原生 DOM 事件：

### 向处理器中传入一个 $event 变量

``` html
<button @click="warn('message1', $event)">
Submit
</button>
```

### 使用内联箭头函数

``` html
<button @click="(event) => warn('message1', event)">
Submit
</button>
```

``` js
function warn(message, event) {
  // 在此处可以访问原生事件
  if (event) {
    event.preventDefault()
  }
  alert(message)
}
```

## 事件修饰符

- stop：调用 event.stopPropagation()，阻止事件继续传播。
- prevent：调用 event.preventDefault()，阻止事件的默认行为。
- self：只当事件是从原始目标元素本身触发时触发回调，如果事件是从内部元素冒泡上来的则不触发。
- capture：添加事件监听器时使用 capture 模式，即在捕获阶段触发而不是冒泡阶段。
- once：只触发一次事件，之后移除该监听器。
- passive：提升页面滚动性能，告诉浏览器事件处理函数不会调用 event.preventDefault()。

### （1）.stop

``` html
<div @click.stop="handleOuterClick">
  <button @click="handleInnerClick">内部按钮</button>
</div>
```

当内部按钮被点击时，handleOuterClick 不会被触发，因为事件不会继续向外部元素传播。

### （2）.prevent

``` html
<form @submit.prevent="handleSubmit">
  <button type="submit">提交</button>
</form>
```

handleSubmit 方法中的 event.preventDefault() 将会被调用，从而阻止表单的默认提交行为。

### （3）.self

``` html
<div @click.self="handleClick">只有当点击该div本身时触发</div>
```

只有当点击该 div 元素本身时，handleClick 方法才会被触发；div 内部其他元素冒泡上来的事件不会触发。

### （4）.once

``` html
<button @click.once="handleClick">点击我，只触发一次</button>
```

当按钮被点击后，handleClick 方法只会被触发一次，之后该事件监听器会被移除。

### （5）.capture

``` html
<div @click.capture="handleCaptureClick">捕获阶段触发</div>
```

事件监听器将在捕获阶段（从外向内）触发，而不是在冒泡阶段触发。

### （6）.passive

``` html
<div @touchstart.passive="handleTouchStart">优化页面滚动性能</div>
```

告诉浏览器 handleTouchStart 方法不会调用 event.preventDefault()，从而有助于提升页面滚动的性能。

## 按键修饰符

```@keyup.enter``` 中的 .enter 就是按键修饰符。

### 常规按键

- enter
- tab
- delete (捕获“Delete”和“Backspace”两个按键)
- esc
- space
- up
- down
- left
- right

### 系统按键

系统按键和常规按键不同的是，与 keyup 一起使用时，只有当系统按键被 **按下** 并松开其他键才会触发。

- ctrl
- alt
- shift
- meta

举例：

``` html
<!-- 常规按键 -->
<!-- 当 Alt + Enter 抬起时，触发 submit 事件 -->
<input @keyup.alt.enter="submit" />

<!-- 系统按键 -->
<!-- 当 按下 ctrl 时触发 -->
<div @click.ctrl="doSth">A</div>
<!-- 当 按住 ctrl + 抬起其他键 时触发 -->
<div @keyup.ctrl="doSth">A</div>
```

### .exact 修饰符

.exact 用于确定触发事件的确定组合。

举例：

``` html
<!-- 当抬起 Ctrl 时，即使同时抬起其他系统按键也会触发 -->
<button @keyup.ctrl="doSth">A</button>

<!-- 仅当抬起 Ctrl 且未抬起任何其他键时才会触发 -->
<button @keyup.ctrl.exact="doSth">A</button>

<!-- 仅当没有抬起任何系统按键时触发 -->
<button @keyup.exact="doSth">A</button>
```
