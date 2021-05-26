# 文件注释

统一安装`VScode`的 [koroFileHeader](https://marketplace.visualstudio.com/items?itemName=OBKoro1.korofileheader)插件。

<p>
  <img :src="$withBase('/fileheader.png')" width=300 alt="fileheader">
</p>


## 添加注释

1. **文件头部添加注释：**

快捷键：`window：ctrl + alt + i`，`mac：ctrl + cmd + i`。

2. **在光标处添加函数注释：**

快捷键：`window：ctrl + alt + t`，`mac：ctrl + cmd + t`。


::: warning 警告！ 
请勿随意添加各种奇怪的注释图案，例如`佛祖保佑永无BUG`之类的。
:::

## 注释配置

安装成功后，请对`setting.json`进行配置，你可以在插件界面，点击`⚙️`按钮，找到`Extension Settings`，然后选择对应的设置入口，添加以下内容：

```json
// 全局配置
"fileheader.configObj": {
  "createHeader": false, // 取消自动创建头部注释
  "autoAdd": false // 取消文件没有头部时自动添加注释
},

// 头部
"fileheader.customMade": {
  "Description": "", // 需要手动输入
  "Author": "你的工作邮箱",
  "LastEditors": "你的工作邮箱",
  "Date": "Do not edit",
  "LastEditTime": "Do not edit"
}
```

可以点击[此处](https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE%E5%AD%97%E6%AE%B5)查看所有配置，原则上配置不可自行修改。

其外，`Author`和`LastEditor`必须使用自己的工作邮箱！！！

## 什么时候使用？

不强制所有文件和方法都添加注释，应结合实际情况使用，如下：

1. 某些抽象出来的功能的文件应添加头部注释，以说明用途；
2. 某些核心或重要方法，涉及入参出参的说明，应添加函数注释；
