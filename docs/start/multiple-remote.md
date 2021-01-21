# 多远程仓库

某些项目，代码需要存储到多个远程仓库，例如 Gitee 和 GitLab 各一份，这种情况下，我们应该以其中一个仓库作为主开发仓库，另一个或其他的，作为备份仓库。

**事例：**

以 Gitee 初始项目仓库为例：

1、先在 Gitee 上创建一个空的仓库，并且 copy 仓库 git 地址；

2、回到项目的 bash 上，给项目添加远程仓库，以 Gitee 命名，并复制 git 地址：

```bash
git remote add Gitee https://gitee.com/test-1/demo.git
```

添加后，可以通过`git remote -v`进行查询。

3、把当前的本地主干分支推送到新的远程仓库：

```bash
git push Gitee master
```

接下来，可以通过指定 origin 对仓库代码进行提交和拉取，为了方便，也可以通过修改 git config 添加一下 alias 命令，例如添加`git push all`进行多个仓库推送：

```bash
# 打开 git 配置文件
vi .git/config

# 添加 alias
[remote "all"]
    url = https://github.com/WorkPlusFE/a.git
    url = https://gitee.com/foreverht/b.git
```

保存并退出后，就可以通过`git push all`推送到两个远程仓库，拉取代码也同理。