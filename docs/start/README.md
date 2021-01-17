

# 通过 cli 创建

`@w6s/cli`是团队内部指定使用的 cli 工具，常用于快速创建项目初始架构。更多关于 w6s-cli 的详细介绍，可以查看[w6s-cli](./cli.html)栏目。

## 安装 cli

通过以下命令进行安装：

```bash
yarn global add @w6s/cli
# or
npm install -g @w6s/cli
```

安装成功后，在 shell 中键入`w6s`，即可查看所有功能说明。

## w6s init

当前`w6s-cli`内置两个项目模版，分别是`admin`和`H5`，如其名，admin 为管理后台类型的前端项目模版，而 H5 就是轻应用的前端项目模版。

假设当前在文件目录A， 执行以下命令：

```bash
w6s init project1
```

此时，会要求使用者选择项目模版，通过`上下方向键`选择需要的模版即可。

选择模版后，cli 会自动在 A 目录下创建 project1 文件夹，并把所选的项目模版生成到 project1 文件夹内，在这之前会有一些人机交互，使用者需要输入一些简单的项目信息：

```bash
? 请输入项目名称 project1
? 请输入项目描述 project1
? 请输入项目创建者 hejx
```

项目创建完毕后，会询问是否自动安装依赖及启动服务，然后输入`yes/no`即可。cli 会自动判断你的电脑上所安装的`Node.js包管理器`，优先会使用`Yarn`进行依赖的安装。

如果没有选择自动安装依赖及启动服务，需要进入 project1 文件夹，执行以下命令：

```bash
# 进入目录
cd project1

# 安装依赖
yarn

# 安装成功后，启动服务
yarn serve # yarn dev 同样可用
```
