# 项目架构

初始项目并非是一个完全空白的状态。基于 Vue 的生态，再结合公司项目的业务形态，会默认添加特定的初始化设定。

例如`Admin`模版，将是以 Element-ui 作为基础，初始化后，将自带登录页面、左侧菜单及路由等，类似开源社区上的各类 admin-template。

<p>
  <img :src="$withBase('/admin-template.png')" alt="admin-template">
</p>

## 技术栈

核心技术栈：

* [vue@2.6.11](https://vuejs.org/)
* vue-router@3.1.6
* vuex@3.1.3"
* vue-i18n@8.17.5
* typescript@3.8.3
* axios@0.19.2
* vue-class-component@7.2.3
* vue-property-decorator@8.4.1
* vuex-module-decorators@0.17.0

UI库：

* `admin` [element-ui@2.9.2](https://element.eleme.cn/#/zh-CN)
* `H5` [vant@2.8.1](https://youzan.github.io/vant-weapp/#/intro)

样式：

* [Sass@1.32.8](https://github.com/sass/dart-sass)

::: tip 关于 Dart Sass
从`w6s-cli@2.3.0`版本开始，`node-sass`被移除，转而使用性能更好、安装对环境依赖更少的`dart-sass`。

相关知识点，请查看下方链接：

* [Wht Dart?](https://sass-lang.com/blog/announcing-dart-sass#why-dart)
* [Node Sass to Dart Sass](https://panjiachen.github.io/vue-element-admin-site/zh/guide/advanced/sass.html#node-sass-to-dart-sass)
:::

其他：

* core-js@3.6.5
* @sentry
  * browser@5.15.5
  * integrations@5.15.5
* @w6s
  * sdk
## 目录说明

以`H5`模版为例，当前项目文件结构如下：

```bash
.
├── README.md
├── babel.config.js            --- babel 配置文件
├── commitlint.config.js       --- commitlint 配置文件
├── mock                       --- 模拟 api，mock 服务
│   └── index.js
├── package.json
├── public
│   ├── favicon.ico
│   └── index.html
├── sentry.config.js           --- sentry 配置
├── src
│   ├── App.vue
│   ├── api                    --- api 目录
│   │   └── user.ts
│   ├── assets                 --- 放置静态资源
│   │   └── logo.png
│   ├── components
│   │   └── HelloWorld.vue
│   ├── i18n.ts                --- 国际化配置入口
│   ├── locales                --- 国际化文件存放位置
│   │   └── zh-CN.json
│   ├── main.ts
│   ├── router                 --- 路由配置
│   │   └── index.ts
│   ├── shims-tsx.d.ts
│   ├── shims-vue.d.ts
│   ├── store                  --- vuex 配置
│   │   ├── index.ts
│   │   └── modules
│   │       └── Counter.ts
│   ├── styles
│   │   └── index.scss
│   ├── typings
│   │   └── Common.ts
│   ├── utils
│   │   ├── cordova            --- cordova 相关，引入了 @w6s/codash
│   │   │   └── index.ts
│   │   ├── http               --- http 相关，封装了一些 api 请求方法
│   │   │   └── https.ts
│   │   └── sentry             --- sentry 模块的设置
│   │       └── index.ts
│   └── views                  --- 页面
│       ├── About.vue
│       └── Home.vue
├── stylelint.config.js        --- stylelint 配置
├── tsconfig.json              --- typescript 配置
├── w6s.config.js              --- w6s-cli-script 配置（详情请看 w6s.config.js 栏目）
├── .browserslistrc            --- babel 转换的目标浏览器版本配置文件
├── .editorconfig              --- ide 的一些默认配置，例如缩进为2个空格
├── .env                       --- 环境配置文件（生产配置）
├── .env.development           --- 环境配置文件（开发配置）
└── .eslintrc                  --- eslint 的配置文件
```

## scripts 命令说明

具体可用命令，请在项目的根目录查看`package.json`的`scripts`标签，以下做简单的作用描述：

* `dev` -- 启动服务，用于项目开发
* `serve` -- 同上
* `build` -- 对项目进行打包
* `lint` -- js 代码检测
* `lint:style` -- 样式检测
* `i18n:report` -- 检测 i18n 的配置是否有缺漏或者存在没被使用的属性
* `inspect` -- 打印出项目的 webpack 配置
* `svg` -- 通过`vue-svgicon`，把 svg 图标生成 vue 控件

## w6s-cli-script

在上一步 scripts 命令里，例如`dev`命令，是通过`w6s-cli-script dev`执行的。`w6s-cli-script`实际是对`@vue/cli-service`的上层封装，除了原始的几个命令方法，包括`serve`、`build`、`inspect`及`lint`，`w6s-cli-script`还内置了`i18n:report`和`lint:style`，以及一系列的插件。也就是说，`@vue/cli-service`支持的，`w6s-cli-script`同样支持，例如，添加第三方的 vue-cli 插件。

> 不能通过 vue add plugin 的方式添加，因为某些插件会改变文件内容，例如在 vue.config.js 添加配置

## w6s.config.js

为了让项目看起来更加专业，配置文件被命名为`w6s.config.js`，这个配置文件会被`w6s-cli-script`读取（如果存在的话）。项目创建后，`w6s.config.js`会有一些初始的配置，例如某些插件的初始化配置，同时，`w6s.config.js`的所有设置，实际跟`vue.config.js`是一致的，也就是说，相关的构建配置，可以直接查看 [vue-cli](https://cli.vuejs.org/config/#vue-config-js) 官网。


::: warning 关于配置
所有配置文件，若无特殊原因，请勿随意修改。
:::