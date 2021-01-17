# Git 规范

WIP

## commitlint

借助 [husky](https://github.com/typicode/husky)，我们可以在 git hook 中定义事件，用于进行多种检测。实际在每次代码提交到仓库前，都会执行 js 及样式的规范检测及自动修复。

除此之外，为了更好地管理项目的提交记录，我们还会检测提交代码时输入的`commit message`，使用的是[@commitlint/config-conventional"](https://github.com/conventional-changelog/commitlint)规范。

基本写法：

```bash
type(scope?): subject  #scope is optional
```