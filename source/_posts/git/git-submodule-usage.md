---
title: Git子模块使用
date: 2021-10-11 21:14:07
tags: git
categories: Git
---

在工作中我们经常遇到一个情况，在一个项目中需要包含并使用到另一个项目，比如开发博客时使用到的主题项目，或者是公司业务中需要在多个项目中使用的库。那该如何独立管理这两个项目，并在一个项目中使用另一个项目呢？

Git 通过子模块来解决这个问题。 子模块允许你将一个 Git 仓库作为另一个 Git 仓库的子目录。 它能让你将另一个仓库克隆到自己的项目中，同时还保持提交的独立。

> 注：如果以下有命令报错或无法生效，请检查 Git 的版本。
>
> 可使用 `git update` 或 `git update-git-for-windows` 命令更新 Git 。速度慢的话可以去淘宝镜像 [下载](https://npm.taobao.org/mirrors/git-for-windows/) 。

## 使用子模块

```shell
$ git submodule add [-b <branch>] [-f|--force] [--name <name>] [--reference <repository>] [--depth <depth>] [--] <repository> [<path>]
```

### 初始化子模块

首先，在主项目中执行以下命令：

```shell
$ git submodule add --name matery git@github.com:xiamu33/hexo-theme-matery.git themes/xiamu-matery
Cloning into 'blog/themes/xiamu-matery'...
...
Resolving deltas: 100% (8/8), done.
```

此时，在指定的 `themes` 目录下会生成一个名为 `xiamu-matery` 子模块。

并且在根目录生成了新的 `.gitmodules` 文件，该配置文件保存了项目 URL 取的本地目录之间的映射：

```shell
[submodule "matery"]
  path = themes/xiamu-matery
  url = git@github.com:xiamu33/hexo-theme-matery.git
```

如果有多个子模块，该文件中就会有多条记录。注意该文件也需要纳入 Git 的版本管理并推送至仓库，这样克隆该项目的人才知道去哪获得子模块。

### 克隆含有子模块的项目

当我们克隆一个包含子模块的项目时，项目中会默认包含该子模块目录，但其中没有任何文件：

```shell
$ git clone git@github.com:xiamu33/blog.git
Cloning into 'blog'...
...
Resolving deltas: 100% (90/90), done.

$ ls blog/themes/xiamu-matery/
$
```

你必须执行 `git submodule update --init` 来初始化本地配置并拉取子模块数据（该命令实际上把 `git submodule init` 和 `git submodule update` 合并成了一步）。

项目中有许多子模块的话这样操作未免有些繁琐，如果给 `git clone` 命令传递 `--recurse-submodules` 选项，它就会自动初始化并更新仓库中的每一个子模块， 包括可能存在的嵌套子模块。

```shell
$ git clone --recurse-submodules git@github.com:xiamu33/blog.git
Cloning into 'blog'...
...
Resolving deltas: 100% (90/90), done.
Submodule 'themes/hexo-theme-matery' (git@github.com:xiamu33/hexo-theme-matery.git) registered for path 'themes/xiamu-matery'
Cloning into 'blog/themes/xiamu-matery'...
Submodule path 'themes/xiamu-matery': checked out '8017ee19d9f040607b4ca58286eb46596f773e61'
```

#### 克隆失败

But，由于国内神奇的网络问题或者其他诡异的现象，“偶尔”会克隆失败（如出现 `OpenSSL SSL_read: SSL_ERROR_SYSCALL, errno 10054` 错误时）。可以尝试以下方法：

- 挂梯子；
- 使用 ssh 克隆： `git clone git@github.com:xiamu33/blog.git` ；
- 关闭 ssl 验证： `git config --global http.sslVerify false` 。

如果只成功克隆了主项目，可执行 `git submodule update --init --recursive` 初始化并拉取项目中嵌套的所有子模块。

#### 如何浅克隆子模块

当我们的子模块项目十分庞大且只需要其最近的提交历史时，可以选择浅克隆子模块。只需给 `git submodule update` 传递 `--depth <depth>`  选项：

```shell
$ git submodule add --name matery --depth 1 git@github.com:xiamu33/hexo-theme-matery.git themes/xiamu-matery
```

已有子模块的项目中则执行：

```shell
$ git submodule update --init --depth 1
```

此时，你本地项目的子模块的克隆将作为浅克隆执行（历史深度为1）。如果你想始终浅克隆该子模块，可执行以下命令：

```shell
$ git config -f .gitmodules submodule.<name>.shallow true
```

> `<name>` 为 `git submodule add` 命令中的 `--name` 选项，默认为子模块路径 `<path>`

这将修改 `.gitmodules` 文件：

```shell
[submodule "matery"]
  path = themes/xiamu-matery
  url = git@github.com:xiamu33/hexo-theme-matery.git
  shallow = true
```

## 在包含子模块的项目上工作

### 拉取子模块改动

```shell
$ git submodule update --remote [<path>]
```

Git 默认会尝试更新 **所有** 子模块， 可以传递 `<path>` 更新指定的子模块。

### 拉取整个项目的改动

默认情况下，`git pull` 命令会递归抓取子模块的更改，但并不会更新子模块，需要再执行：

```shell
$ git submodule update --init --recursive
```

如果想自动化次过程，可以给 `git pull` 命令传递 `--recurse-submodules` 选项：

```shell
$ git pull --recurse-submodules
```

如果总是想以 `--recurse-submodules` 拉取，可将 `submodule.recurse` 设置为 `true` 。这会让 Git 为除 `clone` 外所有支持 `--recurse-submodules` 的命令使用该选项。

#### 子模块的URL变动

如果子模块的 URL 发生变动，即与 `.gitmodules` 文件中的 URL 不同。此时拉取子模块会失败，需执行 `git submodule sync` ：

```shell
$ git submodule sync --recursive
$ git submodule update --init --recursive
```

### 发布子模块改动

如果我们同时在主项目和子模块中提交了更改，但忘记了推送子模块的改动，会导致其他开发人员无法更新子模块。为了确保不发生这种情况，可以让 Git 在推送主项目前检查所有子模块是否已推送。

`git push` 命令可以传递 `--recurse-submodules=check|on-demand|only|no` 选项，你也可以执行 `git config push.recurseSubmodules check|on-demand|only|no` 设置默认行为。

- `check` ：Git 会递归检查所有子模块的改动是否已推送，如果有任何未推送的子模块改动，那么 `push` 操作会直接失败；
- `on-demand`：Git 会尝试在推送主项目前推送所有的子模块，如果某个子模块推送失败，那么主项目也会推送失败；
- `only` ：Git 会仅尝试推送子模块，而不会推送主项目；
- `no` ：当 `push` 操作不需要递归子模块时，使用 `no` 参数或传递 `--no-recurse-submodules` 选项可覆盖 `push.recurseSubmodules` 配置。

### 子模块的技巧

#### 子模块遍历

子模块有个 `foreach` 命令，它可以在所有子模块中执行任意命令。如果项目中包含大量子模块，这将会非常有用。

```shell
$ git submodule foreach "git pull"
Entering 'themes/xiamu-matery'
Already up to date.
```

#### 一些有用的别名

当你不想输入十分冗长的命令又不想设置默认选项时，可以设置一些有用的 Git 别名：

```shell
$ git config alias.sdiff "!git diff && git submodule foreach 'git diff'"
$ git config alias.spush "push --recurse-submodules=on-demand"
$ git config alias.supdate "submodule update --remote --merge"
```

## 删除子模块

- `rm -rf themes/xiamu-matery` ：删除项目中的子模块目录；
- `rm -rf .git/modules/matery` ：删除 `.git` 中的子模块目录；
- `vim .gitmodules` ：删除 `.gitmodules` 文件中子模块的条目；
- `vim .git/config` ：删除项目配置中子模块的条目。

当你遇到诸如以下报错时，也可尝试删除并重新初始化子模块。

> `fatal: 'xxx' already exists in the index`
>
> `fatal: 'xxx' already exists and is not a valid git repo`
>
> `fatal: could not get a repository handle for submodule 'xxx'`
