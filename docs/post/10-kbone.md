# kbone，让你的 H5 快速适配小程序

> [岑成威](https://github.com/CENcw) / 2021-5-28

### Kbone 是什么?

[Kbone](https://github.com/Tencent/kbone)是腾讯开源的一个前端工具库，致力于微信小程序和 Web 端同构的解决方案。

> 微信小程序的底层模型和 Web 端不同，我们想直接把 Web 端的代码挪到小程序环境内执行是不可能的。kbone 的诞生就是为了解决这个问题，它实现了一个适配器，在适配层里模拟出了浏览器环境，让 Web 端的代码可以不做什么改动便可运行在小程序里。

### Kbone 和 taro，mpvue，wepy 等框架对比优势

1. 大部分流行的前端框架都能够在 kbone 上运行，比如 Vue、React、Preact 等；
2. 支持更为完整的前端框架特性，因为 kbone 不会对框架底层进行删改（比如 Vue 中的 v-html 指令、Vue-router 插件）；
3. 提供了常用的 DOM/BOM 接口，让用户代码无需做太大改动便可从 Web 端迁移到小程序端；
4. 在小程序端运行时，仍然可以使用小程序本身的特性（比如像 live-player 内置组件、分包功能）；
5. 提供了一些 Dom 扩展接口，让一些无法完美兼容到小程序端的接口也有替代使用方案（比如 getComputedStyle 接口）。

### 原理分析

#### 一、web 端框架基本原理

首先我们来看下普通 Web 端框架，以 Vue 框架为例，一份 Vue 模板对应一个组件，在代码构建阶段编译成调用 Dom 接口的 JS 函数，执行此 JS 函数就会创建出组件对应的 Dom 树，从而渲染到浏览器页面上。

<p>
  <img :src="$withBase('/minPrinciple.png')" alt="test">
</p>

#### 二、业界常规做法

<p>
  <img :src="$withBase('/kbone0.png')" alt="test">
</p>
原理是把代码语法分析一遍，然后将其中的模板部分翻译成对应的跨端需求的模板（微信小程序、支付宝小程序、H5、APP 等）

#### 三、Kbone 的做法

<p>
  <img :src="$withBase('/kbone.jpeg')" alt="test">
</p>

Kbone 是通过提供 适配器 的方式来实现同构，即运行时兼容，而非静态编译。

Kbone 的适配器核心包含两个部分：

miniprogram-render： 仿造 Dom/Bom 接口，构造仿造 Dom 树；

miniprogram-element: 监听仿造 Dom 树变化，渲染到页面，同时监听用户行为，触发事件。

#### 1.仿造 Dom 树

小程序为了安全和性能而采用了双线程的架构，运行用户 JS 代码的逻辑层是一个纯粹的 JSCore，没有任何浏览器相关的实现，所以没有 Dom 接口和渲染到浏览器上的功能。

小程序的渲染原理：小程序的双线程架构，逻辑层会执行用户的 JS 代码进而产生一组数据，这组数据会发往视图层；视图层接收到数据后，结合用户的 WXML 模板创建出组件树，之后小程序再将组件树渲染出来。这里的组件树和 Dom 树很类似，只是它是由官方内置组件或自定义组件拼接而成而不是 Dom 节点。

kbone 就是利用仿造出来的 Dom 树映射到小程序的组件树上

- 如何仿造

利用自定义组件的特性来自己引用自己来进行组装，从而递归创建组件，进而创建出一棵组件树

<p>
  <img :src="$withBase('/kboneCharacteristics.jpeg')" alt="test">
</p>
<p>
  <img :src="$withBase('/kbone-wxs.jpeg')" alt="test">
</p>

递归的终止条件是遇到特定节点、文本节点或者 children 空节点. 然后在创建出组件树后，将 Dom 节点和自定义组件实例进行绑定以便后续的 Dom 更新和操作即可

kbone 这里还对节点数进行了优化: 因为一次性 setData 到视图层，可能会超过 setData 的大小限制（1024kB),所以对 Dom 树按照一定规则进行裁剪，拆分成多棵子树,然后每个自定义组件管理一棵子树, 减少一些自定义组件实例,分批的 setData 到视图层，可以节省开销

#### 2.仿造事件系统

- 原因

  小程序的事件是视图层到逻辑层的通讯方式，事件绑定在组件上，当被触发时，就会执行逻辑层中对应的事件处理函数。

  小程序的捕获冒泡是在视图层 view 端，因此逻辑层在整个捕获冒泡流程中各个节点接收到的事件不是同一个对象，小程序事件的捕获冒泡和阻止冒泡等操作必须在 WXML 模板中生命，无法使用接口实现

- 实现过程

  当自定义组件监听到用户的操作后，就将事件发往仿造 Dom 树，后续自定义组件监听到的同一个事件的冒泡就直接忽略。

```
当触发改节点，仿造Dom树接收到事件后，再进行捕获和冒泡，让事件在各个节点触发
```

<p>
  <img :src="$withBase('/kboneClick.jpeg')" alt="test">
</p>

vue 转小程序的实际操作流程[点击这里](https://wechat-miniprogram.github.io/kbone/docs/guide/tutorial.html#%E7%BC%96%E5%86%99-webpack-%E9%85%8D%E7%BD%AE)


### 结合 Vue 使用

安装 mp-webpack-plugin 插件

```javascript
yarn add mp-webpack-plugin --dev
或者
npm install mp-webpack-plugin --save-dev
```

在 src 目录中新增 main.mp.js 入口文件

```javascript
import Vue from "vue";
import App from "@/App";
import router from "@/router";
import store from "@/store";

// 需要将创建根组件实例的逻辑封装成方法
export default function createApp() {
  // 在小程序中如果要注入到 id 为 app 的 dom 节点上，需要主动创建
  const container = document.createElement("div");
  container.id = "app";
  document.body.appendChild(container);

  Vue.config.productionTip = false;

  return new Vue({
    router,
    store,
    render: (h) => h(App),
  }).$mount("#app");
}
```

在根目录创建 miniprogram.config.js 文件，添加 mp-webpack-plugin 插件配置

```javascript
module.exports = {
  // 页面 origin，默认是 https://miniprogram.default
  origin: "", // 填写项目中的图片资源地址，建议图片资源使用线上地址
  // 入口页面路由，默认是 /
  entry: "/",
  // 页面路由，用于页面间跳转
  router: {
    // 路由可以是多个值，支持动态路由
    index: [],
  },
  // 特殊路由跳转
  redirect: {
    // 跳转遇到同一个 origin 但是不在 router 里的页面时处理方式，支持的值：webview - 使用 web-view 组件打开；error - 抛出异常；none - 默认值；什么都不做，router 配置项中的 key
    notFound: "index",
    // 跳转到 origin 之外的页面时处理方式，值同 notFound
    accessDenied: "index",
  },
  // app 配置，同 https://developers.weixin.qq.com/miniprogram/dev/reference/configuration/app.html#window
  app: {
    navigationStyle: "custom", // 自定义navigation
  },
  // 全局配置
  global: {},
  // 页面配置，可以为单个页面做个性化处理，覆盖全局配置
  pages: {},
  // 优化
  optimization: {
    domSubTreeLevel: 5, // 将多少层级的 dom 子树作为一个自定义组件渲染，支持 1 - 5，默认值为 5

    // 对象复用，当页面被关闭时会回收对象，但是如果有地方保留有对象引用的话，注意要关闭此项，否则可能出问题
    elementMultiplexing: true, // element 节点复用
    textMultiplexing: true, // 文本节点复用
    commentMultiplexing: true, // 注释节点复用
    domExtendMultiplexing: true, // 节点相关对象复用，如 style、classList 对象等

    styleValueReduce: 5000, // 如果设置 style 属性时存在某个属性的值超过一定值，则进行删减
    attrValueReduce: 5000, // 如果设置 dom 属性时存在某个属性的值超过一定值，则进行删减
  },
  // 项目配置，会被合并到 project.config.json
  projectConfig: {
    appid: "", // 填写小程序的AppId
    projectname: "", // 填写小程序的项目名称
  },
  // 包配置，会被合并到 package.json
  packageConfig: {
    name: "", // 项目名称
    description: "", // 描述
    author: "", // 作者信息
  },
};
```

在根目录创建 .env.mp 文件，添加 mp 环境变量

```javascript
NODE_ENV = mp;
```

修改 vue.config.js 文件，添加打包小程序的 webpack 配置

```javascript
const path = require("path");
function resolve(dir) {
  return path.join(__dirname, dir);
}
const webpack = require("webpack");
const MpWebpackPlugin = require("mp-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
  css: {
    extract: true,
  },
  outputDir: process.env.NODE_ENV === "mp" ? "./dist/mp/common" : "./dist/web",
  configureWebpack: {
    resolve: {
      extensions: ["*", ".js", ".vue", ".json"],
      alias: {
        vue$: "vue/dist/vue.esm.js",
        "@": resolve("src"),
      },
    },
  },
  chainWebpack: (config) => {
    if (process.env.NODE_ENV === "mp") {
      config
        .devtool("node")

        .entry("app")
        .clear()
        .add("./src/main.mp.js")
        .end()

        .output.filename("[name].js")
        .library("createApp")
        .libraryExport("default")
        .libraryTarget("window")
        .end()

        .target("web")

        .optimization.runtimeChunk(false)
        .splitChunks({
          chunks: "all",
          minSize: 1000,
          maxSize: 0,
          minChunks: 1,
          maxAsyncRequests: 100,
          maxInitialRequests: 100,
          automaticNameDelimiter: "~",
          name: true,
          cacheGroups: {
            vendors: {
              test: /[\\/]node_modules[\\/]/,
              priority: -10,
            },
            default: {
              minChunks: 2,
              priority: -20,
              reuseExistingChunk: true,
            },
          },
        })
        .end()

        .plugins.delete("copy")
        .end()

        .plugin("define")
        .use(
          new webpack.DefinePlugin({
            "process.env.isMiniprogram": process.env.isMiniprogram, // 注入环境变量，用于业务代码判断
          })
        )
        .end()

        .plugin("extract-css")
        .use(
          new MiniCssExtractPlugin({
            filename: "[name].wxss",
            chunkFilename: "[name].wxss",
          })
        )
        .end()

        .plugin("mp-webpack")
        .use(new MpWebpackPlugin(require("./miniprogram.config.js")))
        .end();
    }
  },
};
```

修改 package.json 中的 scripts 属性，添加用于开发和打包的任务

```javascript
# 开发
"mp-serve": "vue-cli-service build --watch --mode mp"
# 打包
"mp-build": "vue-cli-service build --mode mp"
```

更多使用方法，请查看[Kbone](https://wechat-miniprogram.github.io/kbone/docs/)官网！

### 当前不足

- 缺少第三方插件，不支持 wxs
- 社区不活跃，解决问题慢，维护，贡献者少
- 性能待提高
- [更多限制](https://wechat-miniprogram.github.io/kbone/docs/qa/#%E9%99%90%E5%88%B6)

### 什么时候使用 Kbone

业内其实已经出现了很多关于同构的解决方案了，每个方案都有自己的优劣，不存在能够完美解决所有问题的方案。kbone 也一样，它的优势在上面提到过，而它的不足也是它的实现原理带来的。kbone 是使用一定的性能损耗来换取更为全面的 Web 端特性支持。

所以关于性能方面，如果你对小程序的性能特别苛刻，建议直接使用原生小程序开发；如果你的页面节点数量特别多（通常在 1000 节点以上），同时还要保证在节点数无限上涨时仍然有稳定的渲染性能的话，可以尝试一下业内采用静态模板转译的方案；其他情况就可以直接采用 kbone 了。