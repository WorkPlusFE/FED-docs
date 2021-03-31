# Vite-2.0 使用感受分享

> [叶德权](https://github.com/Edwinye337826) / 2021-3-14

`vite`是尤雨溪开发的一个项目初始化构建工具，一开始是作为Vue项目构建的二代产品，现在已经提供基于Vue, React, Preact, Lit Element等多个项目模板。

## Vite的优点

用 vite 文档上的介绍，它具有以下特点：

- 快速的冷启动
- 即时的热模块更新(HMR)
- 真正的按需编译

## 快速启动

运行vite项目，可以观察到几乎是秒开的，看代码发现与vue-cli不同的关键点是`index.html`入口文件的导入方式
```html
<script type="module" src="/src/main.ts"></script>
```
在script标签里设置`type="module"`，浏览器就可以使用ES Module来导入main.ts的代码。
```ts
import { createApp } from 'vue'
import App from './App.vue'
createApp(App).mount('#app')
```

<img :src="$withBase('/vite-import.png')" alt="TypeScript">

从上图可以看到，vite对模块并没有进行打包处理，而是直接直接引入，对于`.vue`后缀的文件则是通过query参数形式将template，script，style区分开并分别加载各个模块内容。

相比之下webpack启动一个dev serve，需要将所有模块编译，然后打包成一个bundle，最后才会启动服务。而vite可以省略这个打包步骤，当需要使用某个模块时才动态引入，可以说一个项目越复杂、模块越多，vite的优势越明显。

## HMR热更新

热更新的时候，Vite 只需要立即编译当前所修改的文件即可，所以响应速度非常快。

而 Webpack 修改某个文件过后，会自动以这个文件为入口重新 build 一次，所有的涉及到的依赖也都会被加载一遍，所以反应速度会慢很多。

## Build

vite在生产环境下使用rollup进行打包，并且只能在现代浏览器中运行。如果对兼容性有要求，可以使用官方推出的@vitejs/plugin-legacy插件。

```js
import legacy from '@vitejs/plugin-legacy'

export default {
  plugins: [
    legacy({
      targets: ['defaults', 'not IE 11']
    })
  ]
}
```

legacy插件会打包出两套代码，一套面向新式浏览器，另一套则包含各类 polyfill 和语法兼容来面向老式浏览器，同时在 html 中插入根据浏览器 ESM 支持来选择性加载对应包的代码。


## 开始使用

**1. 安装vite**

同其他开发工具一样，vite 提供了用 npm 或者 yarn 一键生成项目：

```js
$ npm init vite-app <project-name>
$ cd <project-name>
$ npm install or yarn
$ npm run dev or yarn dev

# 或者使用yarn

$ yarn create vite-app <project-name>
$ cd <project-name>
$ yarn
$ yarn dev
```

**2. CSS、JSON加载方式**

- **直接导入CSS**

  在Vite中支持对.vue后缀的文件导入 .css 文件将会把内容插入到 `<style>` 标签中，同时也带有 HMR 支持。
  ```ts
  import "../assets/css/hw.css";
  ```
  需要注意的是这种方式导入的css都是全局的，而不仅仅是该组件。在生产环境中最终都会打包成一个style.css。

- **PostCSS**

  Vite 将自动的在 *.vue 文件中的所有 styles 标签 以及所有导入的 .css 文件中应用 PostCSS。只需要安装必要的插件并且在项目根目录下添加 postcss.config.js 配置文件。

- **CSS 预处理器**

  Vite 已经为 .scss, .sass，.less，.styl 和 .stylus 文件提供了内建支持。不需要为他们安装特定的插件，只需要安装相应的预处理器:

  ```js
  $ npm install -D sass
  ```

  需要注意的一点是Vite 单独为 Sass 和 Less 改进了 `@import` 解析，因而Vite 别名也同样受用，而Stylus则会与vite APi冲突，因此不支持使用`@import`。

**3. 项目配置**

  在项目根目录创建一个`vite.config.js`，vite的配置项还是非常多的，说几个跟vue-cli有点差异的配置。

- **项目别名alias**

  在vite1.0中 别名前后都需要加/，resolve也需要解构才能使用，具体代码: 

  ```js
  import { resolve } from 'path'
  export default {
    alias: {
      '/@/': resolve(__dirname, 'src')
    }
  }
  ```

  而在vite2.0这个问题已经修复:

  ```js
  export default defineConfig({
    alias: {
      '@': path.resolve(__dirname, 'src')
    },
  })
  ```

- **http2**

  HTTP 1中，如果想并发多个请求，必须使用多个 TCP 链接，且浏览器为了控制资源，还会对单个域名有 6-8 个的 TCP 链接请求限制； HTTP 2 则可以使用多路复用，代替原来的序列和阻塞机制。所有请求都是通过一个 TCP 连接并发完成。

  ```js
  export default defineConfig({
    server: {
      https: true
    }
  })
  ```

- **环境变量**

  在项目中，我们经常使用环境变量来做运行时代码判断，也依旧是dev serve时使用development，build时使用production。

  不同点在于不再支持`process.env`这样的访问方式，改成`import.meta.env`

  ```js
  - console.log(process.env.VUE_APP_API)
  + console.log(import.meta.env.VUE_APP_API)
  ```

#### 结语

`Vite2.0` 现在已经不再仅仅为vue生态服务，这会让它被更多的人接受和使用。虽然在短时间内 vite 不会替代 webpack，但是我感觉未来还是非常有可能实现的。因此十分推荐
大家在个人项目中使用并深入了解。