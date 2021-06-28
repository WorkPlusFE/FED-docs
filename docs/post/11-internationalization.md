# 前端快速国际化方案

> [黄宝怡](https://github.com/BaoYihahaha) / 2021-6-28

## 国际化背景

前端技术日新月异，技术栈繁多。以前端框架来说有 React, Vue, Angular 等等，再配以 webpack, gulp 等等构建工具去满足日常的开发工作。同时在日常的工作当中，不同的项目使用的技术栈也会不一样。当需要对部分项目进行国际化改造时，由于技术栈的差异，这时你需要去寻找和当前项目使用的技术栈相匹配的国际化的插件工具。

比如：

- vue + vue-i18n
- angular + angular-translate
- react + react-intl
- jquery + jquery.i18n.property

## 需要考虑的问题

**1. 文案翻译**
- 静态文案翻译(前端静态模板文案)
  > 使用i18n国际化插件匹配字典重新赋值（工作量相当大）
- 动态文案翻译(server端下发的动态数据)

**2. 样式**
- 不同语言长度不一样造成的样式错乱

例如：element-ui 中英文切换时，Table 表行、表头错位问题。

```js
//表行、表头错位问题的解决方法：
<el-table-column 
  prop="a" 
  :label="$t('b.c')"
  show-overflow-tooltip="true"
  :width="($t('b.c')).length*18"
>
```

其中：

1. show-overflow-tooltip="true"表示内容超出宽度后省略号表示
2. a表示绑定的数据
3. b.c表示国际化zh.js,en.js中对应显示的表头名称

**3. 时间成本**

对于目前已有的国际化方案，先抛开学习成本不说，立马上手，做一个国际化项目大概需要多久呢？一天？一周？还是一个月？，我的答案是基本在一周以上，对于紧急的项目时间是极其重要的。

由于现有的国际化方案都极其繁琐、枯燥，且随着项目的进展越来越难以维护，尤其是对于多人开发的大型项目。既然不尽如人意，我们不妨探讨一下方便快速的国际化方案。

## 国际化自动化——思路

对于国际化这种重复性的工作，最合适的就是自动化，一切交给程序来解决，岂不是更香？

1. 自动提取语言包文件
2. 自动添加页面翻译
3. 额外的其他工作

## react + [react-i18n-auto](https://github.com/mr18/react-i18n-auto)全自动化——实现

实现方式：react 国际化自动化插件 [react-i18n-auto](https://github.com/mr18/react-i18n-auto)

<p>
  <img :src="$withBase('/internationalization_1.png')" alt="">
</p>

使用安装：

```bash
npm install react-i18n-auto --save-dev
```

第一步：添加babel插件配置（.babelrc添加方式）：

```js
{ 
  "plugins": [
    "@babel/plugin-transform-runtime",
    "react-i18n-auto",
    "..."
  ] 
}
```

第二步：添加自动化配置 i18n.config.js：

```js
const generator = require('react-i18n-auto/generator')
const path = require('path')

generator.gen({
 excluded: /node_modules|output/, // 排除文件选项（默认为：/node_modules/）
 src: path.resolve(__dirname, './code'), // 源文件目录（必选）
 outputPath: path.resolve(__dirname, './output'), // 国际化配置输出目录（必选）
})
```

然后运行 node i18n.config.js 自动生成配置文件，将 localePolyfill.js，localeUtils.js，语言包文件自动生成到outputPath目录下。

> localePolyfill.js暴露全局方法 $AI, $$AI 和全局变量 LOCALE （语言包），LOCALE_VERSION （语言包版本）


第三步：修改 webpack 配置，为每一个 entry 入口添加 localePolyfill.js：

```js
// webpack.config.js
const path = require('path')
module.exports = {
 entry: {
 main: [
  path.resolve(__dirname, './output/localePolyfill.js'),
  path.resolve(__dirname, './src/index.js')
 ],
 ...
 },
```

第四步：修改当前语言（中文无需加载语言包）：

```js
import React from 'react'
import en_US from '../output/en_US/locale'
import localeUtils from '../output/localeUtils'

localeUtils.locale(en_US)
// lolale.js
module.exports = {
 'I_2gaaanh': 'Student',
 'I_2aq02r1': 'Teacher'
}
```

第五步：唯一的额外的工作，动态加载语言包时（如果语言包已提前加载则无需此操作）：

**修改前**

```js
// const.js 
export default Const = {
 SelectOptions:[
 {
  name:'学生',
  value:'student',
 },
 {
  name:'教师',
  value:'teacher',
 },
 ]
}
```

```js
// app.js
import React from 'react'
import Const from './const'

export default class App extends React.Component {
 render () {
 return <select>
  {
  Const.selectOptions.map(item => {
    return <option key={item.value} value={item.value}>
      {item.name}
    </option>
  })
  }
 </select>
 }
}
```

由于const为常量，当语言包LOCALE更新时，const并不会得到更新，需要手动调用$AI，类似的情况都需要手动更新。

**修改后**

```js
import React from 'react'
import Const from './const'

export default class App extends React.Component {
 render () {
 return <select>
  {
    Const.selectOptions.map(item => {
      return <option key={item.value} value={item.value}>
      {$AI(item.$_name, item.name)} {/*唯一需要修改的地方*/}
      </option>
    })
  }
 </select>
 }
}

```

```js
// 编译后的const.js
// 所有的中文对应的字段，自动添加$_前缀，值为对应中文的uuidKey
export default Const = {
  selectOptions: [{
    name: '学生',
    $_name: "I_2gaaanh",
    value: 'student'
    }, {
    name: '教师',
    $_name: "I_2aq02r1",
    value: 'teacher'
  }]
};
```
编译前后代码对照，不污染源码，无痕开发

编译前效果：
```js
export  default  class App extends React.Comment{
  render(){
    let title = '这是翻译的文字'
    return <div title={title}>
      <div title='这是要翻译的标题'>
        我是要翻译的内容
      </div>
    </div>
  }
}
```
编译后效果：
```js
export  default  class App extends React.Comment{
  render(){
    let title = $T('aa8ds','这是翻译的文字')
    return <div title={title}>
      <div title={$T('see23','这是要翻译的标题')}>
        {$T('ae22s','我是要翻译的内容')}
      </div>
    </div>
  }
}
```
自动生成的语言包配置

locale.js
```js
export let locale = {
    "aa8ds":"这是翻译的文字",
    "see23":"这是要翻译的标题",
    "ae22s":"我是要翻译的内容",
}
```
[react-i18n-auto](https://github.com/mr18/react-i18n-auto)实现了对源码的无污染，开发时无感知，只影响打包后的代码，实现全部自动化。只需将locale.js翻译成对应的语言包文件，根据语言加载对应的语言包即可。对于React项目可实现快速完成国际化开发任务。

> 更多使用详情请参考:[react-i18n-auto github主页](https://github.com/mr18/react-i18n-auto) 

## vue + i18n 极速解决方案[vue-swift-i18n](https://vueswifti18n.richieyu.club/guide/)(vscode插件)

### 契子

Vue国际化非常的繁琐:

- 将所有出现的要国际化的汉字复制取名配置；
- 在vue/js文件中找到汉字位置，区分是在template中标签的label或者其他property中，或者{{}}中script，又或者script中的，手动将一层一层的国际化key拷贝，粘贴；
- 就算是相同的汉子，由于在vue中的语法不同，需要重复的拷贝，粘贴；
- 碰到一堆汉字拼接的简直要了老命/(ㄒoㄒ)/~~

### 使用

1.vue-i18n使用很简单： 安装依赖：npm i vue-i18n -S

引入、调用：

```js
import Vue from 'vue'
import VueI18n from 'vue-i18n'

Vue.use(VueI18n)
```

注册i18n实例并引入语言文件，文件格式等下解析

```js
const i18n = new VueI18n({ 
 locale: 'zh', // 定义默认语言为中文 
 messages: {   
    'zh': require('@/assets/languages/zh.json'),   
    'en': require('@/assets/languages/en.json') 
  }
});
```

将i18n注入到vue实例中

```js
new Vue({
   el: '#app',
   router,
   i18n,
   components: { App},
   template: '<App/>'
 }); 
```

1.安装vue-swift-i18n(vscode插件)

商店搜索`vue-swift-i18n`，或者 ctrl+p，输入 ext install RichieChoo. vue-swift-i18n。

2.使用，见下图：

<p>
  <img :src="$withBase('/internationalization_2.gif')" alt="">
</p>

### 功能

1. 检测 vue/js 中的需要国际化的字，自动生成 json 文件；
2. 根据 json 文件检测 vue/js，检测汉字，自动替换成步骤1生成的 json 的key；
3. 检测 vue/js 文件中的已替换的key，展示对应汉字提示弹窗；
4. xxx.json 文件中，生成扁平化的 locales 的 xxx_flat.json 文件，方便取值复制；
5. 在 vue/js 中提供，提供 t、tt、ttt 代码提示。

### 设计

<p>
  <img :src="$withBase('/internationalization_3.png')" alt="">
</p>

> 约定：汉字–汉字开头的连续非空字符串

### 一、Json 生成，更新(Ctrl+Alt+U/Ctrl+Command+U)

<p>
  <img :src="$withBase('/internationalization_4.gif')" alt="">
</p>

::: v-pre
- 汉字检索原则
- 位于`<template></template>`中的汉字，如`<span>汉字123</span>`
- 位于`<template></template>`中的标签属性的汉字，如`<span title="汉字"></span>`
- 位于`<template></template>`中的双花括号之间的汉字，如`<span>{{test ? "汉字" : "中文" }}</span>`
- 位于`<script></script>`中的"与"之间的汉字，'与'之间的汉字
- 过滤单行注释
- 生成更新Json路径配置
- 生成更新原则
- 当json为空或者文件不存在，将检测的汉字当做value，将[modulePrefix].[parents(level读取)].[当前vue文件名字]+唯一Id当做key，存储在json中
- 当json文件不为空，执行智能替换
- 备注：主要是防止国际化后，执行JSON生成命令误操作，会导致json数据为空或错误
- 智能替换：
i. 相同val时，新的key,val替换原来的key,val
ii. 不同val时，保存新增key,val和原有的key,val
::: 
### 二、国际化替换(Ctrl+Alt+I/Ctrl+Command+I)

<p>
  <img :src="$withBase('/internationalization_5.gif')" alt="">
</p>

替换原则
::: v-pre
- 汉字检索原则1，汉字123替换为 {{$t('unique-key')}}
- 汉字检索原则2，title="汉字" 替换为 :title="$t('unique-key')"
- 汉字检索原则3，汉字替换为 $t('unique-key')
- 汉字检索原则4，汉字替换为 this.$t('unique-key')
::: 

### 三、国际化提示(Ctrl+Alt+O/Ctrl+Command+O)

1.提示原则
  - 正则：/(?<=\$t\(["'])[^'"]+/gm 匹配已替换的字符串
  - 用新生成的唯一key而不是json的key来标识，为了防止json中的key被使用多次

2.提示依据Json

### 四、Json 扁平处理

<p>
  <img :src="$withBase('/internationalization_6.gif')" alt="">
</p>

1.扁平化原则：
  - 将所有的有value的key的所有父对象和key用.连接
  - Json扁平处理没有提供快捷键，通过右键文件夹或者json文件来执行命令

2.扁平依据选中json，生成/更新xxx_flat.json与json文件路径同级

### 五、路径及 JSON

```
根目录：认定当前项目package.json为根目录
当前文件：执行Json生成等命令所在的文件
```

1.路径
  - 默认路径：[根目录]/src/locales/zh-cn.json 为默认json路径
  - 提供字符串配置项：Default Locales Path,如"test",则对应的json路径：[根目录]/``test/zh-cn.json

2.json文件的属性名及value
  - 默认：[当前文件的父文件夹名].[当前文件名(无后缀)]
  - 提供数字层级配置项：Parent Dir Level,如3则代表属性名头部添加取3层父文件夹名
  - 提供字符串配置项：**Module Prefix Fo Update JSON**，如 “sdm-ui”，会把“sdm-ui"添加到父文件夹名之前
  - 其他配置项：
    - Not Alert Before Update I18n，默认提示，若为true则会直接更新- json不弹窗提醒
    - Do Not Disturb,默认false,若为true则会关闭任何命令提醒
    - I18n Value Hover，默认true,开启悬浮提示框功能

### 六、代码提示
::: v-pre

1. 汉字检索原则1，tt 替换为 {{$t('剪切板内容')}}
2. 汉字检索原则2，t 替换为 $t('剪切板内容')，需手动加：
3. 汉字检索原则3，t 替换为 $t('剪切板内容')
4. 汉字检索原则4，ttt 替换为 this.$t('剪切板内容')
::: 


更多使用方法，请查看[Vue Swift I18n](https://vueswifti18n.richieyu.club/guide/)官网！