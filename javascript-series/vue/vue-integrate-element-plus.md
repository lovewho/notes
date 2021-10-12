# Using Vue-3 to integrate element-plus

## 1. Introduction

> 本节我们将学习如何使用 Vue 整合 element-plus。
>
> 默认使用 vue-cli 创建的项目还是基于 Webpack 的，后续将使用 Vite。
>
> [Here is element-plus official site](https://element-plus.gitee.io/zh-CN/)
>
> [Here is Vue CLI official site](https://cli.vuejs.org/)

## 2. To Integrate element-plus

- 创建一个 Vue-3 的项目:

  ```bash
  $ vue create vue-with-element-plus
  > Default ([Vue 2] babel, eslint)
  > Default (Vue 3) ([Vue 3] babel, eslint):`[this]`
  > Manually select features
  ```

- 启动项目并通过浏览器访问：

  ```bash
  $ npm run serve
  > Local:   http://localhost:8080/
  > Network: http://192.168.x.x:8080/
  ```

- 安装 element-plus 模块，作为生产模块:

  ```bash
  $ npm install element-plus -P
  ```

## 3. Usage

该节介绍如何导入 element-plus 并使用它。

### 3.1. Import

#### 3.1.1. Full Import

如果您不担心包的大小，使用全量导入会更方便：

```js
//main.js
import { createApp } from 'vue'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import App from './App.vue'

createApp(App).use(ElementPlus).mount('#app')
```

#### 3.1.2. On-demand Import

您也可以按需引入组件，这样只会导入您指定的组件：

```javascript
//main.js
import { createApp } from 'vue'
import App from './App.vue'
// 导入 el-button 组件
import { ElButton } from 'element-plus'
// 导入组件的样式
import 'element-plus/es/components/button/style/css'

const app = createApp(App)
app.use(ElButton)
app.mount('#app')
```

但这种方式有一个很明显的缺点，就是每引用一个组件您都必须引入对应的 CSS 样式。值得高兴是，有这样的插件可以帮我们自动导入，它就是 [unplugin-vue-components](https://github.com/element-plus/unplugin-element-plus#readme)。

首先您需要安装该插件，作用范围(开发环境)：

```bash
$ npm install unplugin-vue-components -D
```

然后您需要在 webpack 配置文件中添加如下代码，在这里就是 *vue.config.js* 文件中：

```javascript
//vue.config.js
const Components = require('unplugin-vue-components/webpack')
const { ElementPlusResolver } = require('unplugin-vue-components/resolvers')

module.exports = {
  	//...
    configureWebpack: {
        plugins: [
            Components({
                resolvers: [ElementPlusResolver()],
            }),
        ]
    }
    //...
}
```

接下您只需引入 element-plus 组件即可，该插件会帮您自动引入样式：

```javascript
//main.js
import { createApp } from 'vue'
import App from './App.vue'
// 导入 el-button 组件
import { ElButton } from 'element-plus'

createApp(App).use(ElButton).mount('#app');
```

需要注意的一点是，在 main.js 导入的组件在所有页面都可用，如果某些组件您只想在特定页面可用，那么您应该在该页面引入组件而不是 main.js。像这样：

```vue
//App.vue
<template>
  <el-button>element-plus</el-button>
</template>

<script>
import { ElButton } from 'element-plus'

export default {
  name: 'App',
  components: {
    ElButton
  }
}
</script>
```

### 3.2. Use it

成功安装和导入 element-plus 后，您就可以在页面中去使用它啦。

```vue
<template>
    // 使用 el-button 组件
  <el-button>element-plus</el-button>
</template>
```

运行项目您应该可以在页面看到一个按钮：

<img src="../../assets/javascript/vue-element-plus-button.png" style="zoom:50%;" />

## 4. Global Configuration

当导入 element-plus 时，您可以传递一个全局配置对象用于设置表单组件的默认大小，以及弹出组件的 zIndex，zIndex 的默认值为 2000。

**Full import:**

```javascript
import { createApp } from 'vue'
import ElementPlus from 'element-plus'
import App from './App.vue'

const app = createApp(App)
// 组件
app.use(ElementPlus, { size: 'small', zIndex: 3000 })
```

**On-demand:**

```javascript
import { createApp } from 'vue'
import { ElButton } from 'element-plus'
import App from './App.vue'

// 配置组件的属性
const app = createApp(App);
app.config.globalProperties.$ELEMENT = {
   // options
   size: 'mini' 
}
app.use(ElButton).mount('#app')
```

## 5. Conclusion

本节我们学习了如何使用 Vue 整合 Element Plus。并介绍了：

- 全量导入和按需导入
- 使用插件自动导入 CSS 样式
- 组件的全局配置











