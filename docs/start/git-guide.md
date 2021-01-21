# 代码提交规范
## commit message 规范

借助 [husky](https://github.com/typicode/husky)，我们可以在 git hook 中定义事件，用于进行多种规范检测。

为了更好地管理项目的提交记录，目前通过 w6s-cli 创建的项目，已经具备检测 commit message 的功能，我们会检测提交代码时输入的`commit message`，使用的是[@commitlint/config-conventional"](https://github.com/conventional-changelog/commitlint)规范。

基本写法：

```bash
type(scope?): subject  #scope is optional
```

**`type`类型如下：**

type 代表某次提交的类型，比如是修复一个 bug 或增加一个新的 feature。

* `feat` 新增功能；
* `fix` 修复 bug；
* `docs` 仅仅修改了文档，比如 README, CHANGELOG, CONTRIBUTE 等等；
* `style` 仅仅修改了空格、格式缩进、逗号等等，不改变代码逻辑；
* `refactor` 代码重构，没有加新功能或者修复 bug；
* `perf` 优化相关，比如提升性能、体验；
* `chore` 改变构建流程、或者增加依赖库、工具等；
* `revert` 回滚到上一个版本。

**可选的 scope：**

`scope` 表示一个范围，例如一个项目里面可能会同时包含多个入口，admin、mobile等，可以通过 scope 进行区分，如：

```bash
feat(admin): Add Excel import feature
```

**subject：**

尽可能简短地描述任务内容，并尽量使用中文。

## 提交频率

日常开发过程中，尽量保持提交的粒度为一个 feature 或修复一个 bug。

例如，你在一个小时内，修复了多个简单的 bug，那应该分多次提交，而且为了避免修复的 bug 可能存在于同一个代码文件，应修复后及时提交。

再者，如果一个 feature 需要开发多天，那至少保证每天提交或合并一次，避免代码差异增大。