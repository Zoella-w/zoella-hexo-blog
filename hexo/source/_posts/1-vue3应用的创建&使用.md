---
title: vue3 应用的创建&使用
date: 2023-11-06 11:27:47
tags:
    - vue3
    - vue
categories:
    - vue3
      - 文档
---

## 创建应用

``` bash
npm create vue@latest
```

## 通过CDN使用vue

这里使用了提供 npm 包服务的 CDN —— unpkg

``` html
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
```

### DOM 中的根组件模板

当根组件没有设置 template 选项时，Vue 将自动使用容器的 innerHTML 作为模板

这种方式通常用于此种“无构建步骤”（通过 CDN 使用 vue）的应用程序

``` html
<div id="app">
  <button @click="count++">{{ count }}</button>
</div>
```

``` js
import { createApp } from 'vue';

const app = createApp({
  data() {
    return {
      count: 0
    }
  }
}).mount('#app');
```

### 全局构建

上述链接使用 *全局构建* 版本的 Vue，所有顶层 API 都暴露为全局 Vue 对象的属性

``` html
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<div id="app">{{ message }}</div>

<script>
  const { createApp, ref } = Vue;
/**  1. 创建应用并挂载 */
  createApp({
    setup() {
      const message = ref('Hello world!');
      return {
        message
      };
    }
  }).mount('#app');
  
  /**  2. 分步骤 */
  // // a. 创建应用
  // const app = createApp({
  //   setup() {
  //     const message = ref('Hello world!');
  //     return {
  //       message
  //     }
  //   };
  // });
  // // b. 挂载
  // app.mount('#app');
</script>
```

### 导入映射表（Import maps）

不使用完整的 CDN URL 引入，而使用 es6 的导入映射表（import maps）

``` html
<script type="importmap">
  {
    "imports": {
      "vue": "https://unpkg.com/vue@3/dist/vue.global.js"
    }
  }
</script>
<div id="app">{{ message }}</div>

<script type="module">
  import { createApp, ref } from "vue";

  createApp({
    setup() {
      const message = ref("Hello world!");
      return {
        message
      };
    }
  }).mount('#app');
</script>
```

### 拆分模块

将代码拆分为单独的 js 文件以便管理

出于安全原因，es 模块只能通过 http 协议工作，所以需要使用本地的 http 服务器，通过 http 协议提供 index.html。比如：安装 Node.js，在 html 文件所在的文件夹运行 ``npx serve``

``` html
<!-- index.html -->
<div id="app"></div>

<script type="module">
  import { createApp } from 'vue';
  import MyComponent from './my-component.js';
  createApp(MyComponent).mount('#app');
</script>
```

``` js
// my-component.js
import { ref } from 'vue';

export default {
  setup() {
    const count = ref(0);
    return { count };
  },
  template: `<div>count is {{ count }}</div>`
}
```
