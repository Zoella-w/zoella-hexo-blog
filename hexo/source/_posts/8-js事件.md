---
title: js 事件
date: 2023-11-29 17:38:02
tags:
    - JavaScript
categories:
    - JavaScript
      - 事件
---

事件捕获和事件冒泡 是浏览器处理DOM元素事件的两种方式（顺序：先捕获，再冒泡）。

``` html
<!DOCTYPE html>
<html>
  <body>
      <div>click me</div>
  </body>
</html>
```

## 事件捕获

事件捕获从文档根节点开始，逐级向下传播到目标元素。

点击 div，事件传播方向：document -> html -> body -> div

## 事件冒泡

事件冒泡指当目标元素触发了某事件时，事件会从该元素开始逐级向上传播，直到文档根节点。

点击 div，事件传播方向：div -> body -> html -> document

## 事件模型

### DOM0 事件模型

DOM0 只在冒泡阶段处理事件处理程序。

``` html
<div onclick="handleClick">click</div>
<script>
  function handleClick() {
    alert('clicked');
  }
</script>
```

### DOM2 事件模型

DOM2 在捕获阶段和冒泡阶段都可以处理事件处理程序（更灵活可控）。

使用：addEventListener() 和 removeEventListener()

``` html
<div id="myBtn">click</div>
<script>
  function handleClick() {
    alert('clicked');
  }
  const btn = document.getElementById('myBtn');
  btn.addEventListener('click', handleClick);
</script>
```

### IE 事件模型

功能类似于 DOM0，使用方式上类似于 DOM2。

使用 attachEvent() 和 detachEvent() 方法。

``` html
<div id="myBtn">click</div>
<script>
  function handleClick() {
    alert('clicked');
  }
  const btn = document.getElementById('myBtn');
  btn.attachEvent('click', handleClick);
</script>
```

## js 实现事件修饰符

### 阻止默认行为 preventDefault

比如阻止链接的跳转或表单的提交

``` js
document.getElementById('btn').addEventListener('click', (event) => {
  event.preventDefault();
});
```

### 阻止事件冒泡 stopPropagation()

阻止事件传到父元素

``` js
document.getElementById('btn').addEventListener('click', (event) => {
  event.stopPropagation();
});
```

### 一次性事件处理

``` js
function handleClick(event) {
  this.removeEventListener('click', handleClick); // 点击一次移除监听器
}
document.getElementById('btn').addEventListener('click', handleClick);
```

### 获取键盘按键

``` js
document.addEventListener('keydown', (event) => {
  console.log(event.key);
});
```

### 获取鼠标按键

``` js
document.addEventListener('mousedown', (event) => {
  console.log(event.key);
});
```
