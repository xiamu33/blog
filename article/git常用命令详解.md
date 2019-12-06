# git 常用命令

## 运行前配置

### 配置文件

git 自带了一个 `git config` 工具来帮助设置 git 外观和行为的配置。这些配置分别存储在三个不同的位置：

1. `${prefix}/etc/gitconfig` 文件：系统上每个用户及其仓库的通用配置。使用 `git config --system` 选项时，会从该文件读写配置。

2. `~/.gitconfig` 或 `~/.config/git/config` 文件：只针对当前用户。通过 `git config --global` 读写该文件。

3. `.git/config` 文件：即当前仓库 git 目录中的配置文件，仅针对该仓库。通过 `git config --local` 读写该文件。

配置优先级依次为：3 > 2 > 1。

通过 `git config --list [--system|--global|--local]` 命令来查看对应的配置变量。

你可能会看到重复的变量名，因为 git 会从不同文件中读取同一个配置。这种情况下，git 会使用它找到的每一个相同变量的最后一个配置。

**若无法在对应目录找到配置文件，可执行 `git config --list --show-origin` 来查看每个配置变量所在文件。**

> 注：[]中为可选项，下同。

### 用户配置

安装完 git 后第一件事就是设置你的用户名和邮箱地址。因为每次 git 提交都会使用这些信息，并且会写入每次的提交中。

```javascript
$ git config --global user.name "xiamu"
$ git config --global user.email "xiamu@example.com"
```

### 获取帮助

当使用 git 时需要获取帮助时，有三种方法可以找到 git 命令的使用手册：

```javascript
$ git help <verb>
$ git <verb> --help // 这里一定要用--help，使用-h会被git理解为输入选项
$ man git-<verb> // 这个方法无法查询git别名
```

> 例如：

```javascript
$ git help clone

GIT-CLONE(1)                                         Git Manual                                         GIT-CLONE(1)



NAME
       git-clone - Clone a repository into a new directory

SYNOPSIS
       git clone [--template=<template_directory>]
                 [-l] [-s] [--no-hardlinks] [-q] [-n] [--bare] [--mirror]
                 [-o <name>] [-b <name>] [-u <upload-pack>] [--reference <repository>]
                 [--dissociate] [--separate-git-dir <git dir>]
                 [--depth <depth>] [--[no-]single-branch] [--no-tags]
                 [--recurse-submodules[=<pathspec>]] [--[no-]shallow-submodules]
                 [--jobs <n>] [--] <repository> [<directory>]
...
```

```javascript
$ git co --help // 这里'co'被事先定义成了'checkout'的别名

'co' is aliased to 'checkout'
```

```javascript
$ man git-checkout // 正常提示

GIT-CHECKOUT(1)                                      Git Manual                                      GIT-CHECKOUT(1)



NAME
       git-checkout - Switch branches or restore working tree files

SYNOPSIS
       git checkout [-q] [-f] [-m] [<branch>]
       git checkout [-q] [-f] [-m] --detach [<branch>]
       git checkout [-q] [-f] [-m] [--detach] <commit>
       git checkout [-q] [-f] [-m] [[-b|-B|--orphan] <new_branch>] [<start_point>]
       git checkout [-f|--ours|--theirs|-m|--conflict=<style>] [<tree-ish>] [--] <paths>...
       git checkout [<tree-ish>] [--] <pathspec>...
       git checkout (-p|--patch) [<tree-ish>] [--] [<paths>...]
...
```

```javascript
$ man git-co // 无法识别git别名

No manual entry for git-co
```

## git 基础

### 获取 git 仓库

#### 克隆现有仓库

```javascript
$ git clone git@github.com:xiamu33/my-project.git my-local-project
```

该命令默认会在本地当前目录下创建一个名为 `my-local-project` 的目录（如果省略该选项，则目录名为`my-project` ），并初始化一个 `.git` 文件夹，并从远程仓库拉取所有数据放入 `.git` 文件夹，然后从中读取最新版本的文件拷贝。

git 克隆的是该 git 仓库上几乎所有的数据。如果你的服务器磁盘坏了，通常可以使用任何一个克隆下来的用户端来重建服务器上的仓库。

#### 在现有目录中初始化仓库

在项目根目录中执行：

```javascript
$ git init
```

该命令会创建一个名为 `.git` 的子目录，这个子目录中含有初始化的 git 仓库中所有的必须文件。

#### 总结

- `git clone <url> [<local-project-name>]` 从远程服务器克隆仓库至本地目录名下。

- `git init` 初始化本地仓库。

### 远程仓库使用

#### 查看远程仓库

在已跟踪远程仓库的本地仓库目录中执行 `git remote [show]` 命令，会列出你指定的远程服务器的简写。

```javascript
$ git remote // 远程仓库服务器的默认名字即为origin 选项show可以省略

origin
```

指定选项 `-v` 或 `--verbose` ，会显示需要读写远程仓库使用的 git 保存的简写及其对应的 url。

```javascript
$ git remote -v

origin  git@github.com:xiamu33/blog.git (fetch)
origin  git@github.com:xiamu33/blog.git (push)
```

#### 添加远程仓库

执行 `git remote add <short-name> <url>` 添加一个新的远程 git 仓库。

```javascript
$ git remote
origin

$ git remote add repos https://github.com/xiamu33/blog.git

$ git remote -v
origin  git@github.com:xiamu33/blog.git (fetch)
origin  git@github.com:xiamu33/blog.git (push)
repos   https://github.com/xiamu33/blog.git (fetch)
repos   https://github.com/xiamu33/blog.git (push)
```

#### 重命名与删除远程仓库

执行 `git remote rename <old-name> <new-name>` 可修改一个远程仓库的简写名。

```javascript
$ git remote rename repos https

$ git remote
origin
https
```

值得注意的是，这样也会修改远程分支的名字，原来跟踪 `repos/master` 的分支现在会跟踪 `https/master` 。

执行`git remote rm <remote-name>` 可删除一个远程仓库。

```javascript
$ git remote rm https

origin
```

#### 从远程仓库抓取与拉取

执行`git fetch [<remote-name>]` 可从远程仓库中获得数据。

`git fetch` 命令会将数据抓取到你的本地仓库，但不会自动合并或修改你当前的工作。你必须手动将其合并入你的工作。

如果你的本地分支已跟踪一个远程分支，执行 `git pull` 命令来自动拉取然后合并远程分支到当前分支。

或者执行`git pull [<remote-name> <remote-branch-name>[:<local-branch-name>]]` 拉取指定远程分支，省略本地分支`<local-branch-name>`选项则拉取至远程分支同名分支，若无本地同名分支则新建该分支。

```javascript
$ git pull https master:dev

From https://github.com/xiamu33/blog
 * [new branch]      master     -> dev
Already up to date.

$ git branch -vv // 注意：新建的本地分支并不会自动跟踪远程分支
  dev    363cfc9 first commit
* master 363cfc9 [origin/master] first commit
```

另外，可执行`git branch --set-upstream-to=<remote-name>/<branch-name>` 让本地分支**跟踪远程分支**，之后可直接执行`pull` `push` 拉取或推送分支。

#### 推送至远程仓库

执行`git push [<remote-name> <local-branch-name>[:<remote-branch-name>]]` 可以将本地分支推送至指定远程仓库的指定分支。

若省略`<remote-branch-name>` 选项，则推送至本地分支同名远程分支，若指定本地不存在，则会报错；若指定远程分支不存在，则会创建一个远程分支。

```javascript
$ git push https master2:master
error: src refspec master2 does not match any.
error: failed to push some refs to 'https://github.com/xiamu33/blog.git'

$ git push https master:dev
Total 0 (delta 0), reused 0 (delta 0)
remote:
remote: Create a pull request for 'dev' on GitHub by visiting:
remote:      https://github.com/xiamu33/blog/pull/new/dev
remote:
To https://github.com/xiamu33/blog.git
 * [new branch]      dev -> dev

$ git branch -r // 查看远程分支 同git branch --remotes
https/dev
https/master
origin/master
```

如果你的本地分支已跟踪一个远程分支，可省略后面的选项执行 `git push` 推送至远程仓库。

#### 总结

- `git remote [show]` 查看远程仓库的简写。

- `git remote -v` 查看远程仓库的简写及其对应的 url。

- `git remote add <short-name> <url>` 添加远程仓库。

- `git remote rename <old-name> <new-name>` 重命名远程仓库（也会修改远程分支的名字）。

- `git remote rm <remote-name>` 删除远程仓库。

- `git fetch [<remote-name>]` 从远程仓库抓取数据（不会合并入本地工作）。

- `git pull [<remote-name> <remote-branch-name>[:<local-branch-name>]]` 拉取指定远程分支数据至指定（或同名）本地分支，省略所有选项则拉取已跟踪远程分支数据。

- `git branch --set-upstream-to=<remote-name>/<branch-name>` 让本地分支跟踪远程分支。

- `git push [<remote-name> <local-branch-name>[:<remote-branch-name>]]` 推送至远程仓库。

- `git branch -r` 查看远程分支。

### 记录每次更新

git 文件的生命周期如下图：

![lifecycle](../img/lifecycle.png)

#### 查看当前文件状态

执行`git status` 可查看当前文件的详细状态。

```javascript
$ git status

On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

       modified:   .markdownlint.json
       modified:   article/git常用命令详解.md
       new file:   img/lifecycle.png

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

       modified:   README.md
       modified:   article/git常用命令详解.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)

       .gitignore
```

执行`git status -s` 或 `git status --short` 将输出更为紧凑的状态信息。

```javascript
$ git status -s

M  .markdownlint.json
 M README.md
MM article/git常用命令详解.md
A  img/lifecycle.png
?? .gitignore
```

- `??` 标记：新添加的未跟踪的文件。

- `A` 标记：新添加到暂存区中的文件。

- `M` 标记：修改过的文件。出现在右表示该文件被修改了但未放入暂存区；出现在左表示该文件被修改了并且放入了暂存区；左右都出现表示该文件被修改了，一部分修改已被放入暂存区，另一部分还没有。

- `D` 标记：已删除的文件。

#### 跟踪暂存文件

执行 `git add <file-name>` 跟踪一个新文件。

```javascript
$ git add README.md
```

这将会跟踪 `README.md` 文件，并暂存至暂存区。

`git add` 如果使用目录的路径作为参数，该命令将递归地跟踪该目录下的所有文件。如：`git add .` 会跟踪并暂存当前目录层级下所有文件（被忽略的文件除外）。

#### 查看修改

通过 `git status` 我们可以查看当前哪些更新文件还没有暂存，哪些文件已经暂存准备下次提交，但只限文件名。

知道具体修改了什么地方，可以执行 `git diff` 命令，该命令将通过文件补丁的格式显示具体哪些行发生了改变。

编辑的文件不暂存，执行 `git status` 命令会看到：

```javascript
$ git status

On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

       modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
```

想要查看尚未暂存的文件更新了哪些部分，不加参数直接输入 `git diff` ：

```javascript
$ git diff

diff --git a/README.md b/README.md
index a093180..cba457d 100644
--- a/README.md
+++ b/README.md
@@ -5,3 +5,5 @@
 ## git 相关
+
+- [git 常用命令详解](./article/git常用命令详解.md)
```

该命令比较的是工作目录中当前文件和暂存区域快照之间的差异， 也就是修改之后还没有暂存起来的变化内容。

若要查看已暂存的将要添加到下次提交的更改，可以执行 `git diff --cached` 命令（或者 git >= 1.6.1 版本所提供的 `git diff --staged` ）。

**需要注意的是，`git diff` 本身只显示已跟踪文件中尚未暂存的改动，而不是自上次提交以来所做的所有改动。**

> `git difftool` 命令可以将文件差异通过图形化的方式输出结果，使用文件名或目录路径作为参数，可以查看指定文件的改动。执行 `git difftool --tool-help` 命令查看你的系统支持哪些 Git Diff 插件。

#### 提交更新

当所有修改的文件都 `git add` 到暂存区准备提交后，就可以执行 `git commit` 命令。

不加任何参数执行 `git commit` 命令会启动文本编辑器以便输入本次提交的说明，一般都是 vim 或 emacs。也可以执行 `git config --global core.editor` 命令设定编辑软件。

```javascript
$ git commit

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch master
# Your branch is up to date with 'origin/master'.
#
# Changes to be committed:
#       modified:   article/git常用命令详解.md
#
~
~
"~/Documents/blog/.git/COMMIT_EDITMSG" 13L, 367C
```

默认的提交消息有一段注释，并包含最后一次执行 `git status` 的输出。如果想要知道更详细的修改提示，可以使用 `-v` 选项。输入完提交说明退出编辑器时，git 会丢掉注释行，用你输入的提交信息生成一次提交。

另外，可以使用 `-m` 选项，后面加上提交说明，可以直接生成一次提交。

```javascript
$ git commit -m "add git: 基础用法更新"

[master 2f2a8fa] add git: 基础用法更新
 1 file changed, 88 insertions(+)
```

现在已经成功创建一个提交。提交后的输出会告诉你，当前是在哪个分支提交的（`master`），本次提交的部分 SHA-1 校验和是什么（`2f2a8fa`），本次提交有多少文件修订过，多少行添加和删改过。

请记住，提交时记录的是放在暂存区域的快照。 任何还未暂存的仍然保持已修改状态，可以在下次提交时纳入版本管理。 每一次运行提交操作，都是对你项目作一次快照，以后可以回到这个状态，或者进行比较。

##### 跳过使用暂存区域

尽管使用暂存区域的方式可以精心准备要暂存的细节，但每次都这么操作也略显繁琐。git 提供了一个跳过使用暂存区域的方式，就是在 `git commit` 后面加上 `-a` 选项（需在 `-m` 之前），这样 git 就会自动把修改中已经跟踪过的文件暂存起来一并提交，跳过手动 `git add` 步骤。

```javascript
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")

$ git commit -a -m "modify README"
[master 1d5ef89] modify README
 1 file changed, 2 insertions(+)
```

需要注意的是，新添加的文件（即之前未跟踪的文件）并不会被自动暂存起来，仍需要手动 `git add` 。觉得繁琐可以通过设置 git 别名快速提交，git 别名会在后面详细提到。

#### 忽略文件

一般我们总有些文件不想纳入 git 管理，也不希望它们总出现在未跟踪文件列表。比如日志文件，或者编译过程中创建的临时文件等。在这种情况下，我们可以创建一个名为 `.gitignore` 的文件，列出要忽略的文件模式。

文件 `.gitignore` 的格式规范如下：

- 所有空行或者以`＃`开头的行都会被 Git 忽略。

- 可以使用标准的 glob 模式匹配。

- 匹配模式可以以（`/`）开头防止递归。

- 匹配模式可以以（`/`）结尾指定目录。

  - 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（`!`）取反。

所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。

`*` 匹配零个或多个任意字符；

`[abc]` 匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）。如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如`[0-9]`表示匹配所有 0 到 9 的数字）。

`?` 只匹配一个任意字符；

`**` 表示匹配任意中间目录，比如`a/**/z`可以匹配`a/z`,`a/b/z`或`a/b/c/z`等。

如果有一个文件你不想忽略却意外被 git 忽略了，这时可以执行 `git check-ignore` 命令检查被那条规则忽略了。选项 `-v` 可以输出匹配模式的详细信息。

```javascript
$ git check-ignore -v .DS_Store

.gitignore:1:.DS_Store  .DS_Store
```

该输出说明 `.DS_Store` 文件是被 `.gitignore` 文件中第一行的 `.DS_Store` 规则忽略的。

#### 移除/移动文件

##### 移除文件

如果要从 git 中移除某个文件，就必须要从已跟踪文件清单中移除（确切地说，是从暂存区域移除），然后提交。可以执行 `git rm` 命令完成此项工作，并连带从工作目录中删除指定的文件，这样以后就不会出现在未跟踪文件清单中了。

但是如果只是单纯的从工作目录中删除文件，执行 `git status` 时会显示“Changes not staged for commit”（未暂存）。

```javascript
$ rm README.md
$ git status

On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    README.md

no changes added to commit (use "git add" and/or "git commit -a")
```

这时候需要运行 `git rm` 或者 `git add` 操作记录移除文件操作。

```javascript
$ git rm README.md
rm 'README.md'

$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        deleted:    README.md
```

下一次提交时，该文件就不再纳入版本管理了。如果删除之前修改过并且已经放到暂存区域的话，想要强制删除则必须要用强制删除选项 `-f` 了，但是这样没有添加到快照中的数据无法被 git 恢复。

##### 仅从暂存区移除

当你想把文件从 git 仓库中删除（即让 git 取消跟踪），但是仍要保留在当前目录中。比如在忘记添加 `.gitignore` 文件时，将一堆日志文件添加到了暂存区，这时候可以使用 `--cached` 选项。

```javascript
$ git rm --cached README.md
```

`git rm` 命令后面可以列出文件或者目录的名字，也可以使用`glob`模式：

```javascript
$ git rm log/\*.log
$ git rm --cached log/\*.log
```

注意星号 `*` 前的 `\` 符号， 因为 git 有自己的文件模式扩展匹配方式，所以不需要用 shell 来帮忙展开（当然不加 `\` 符号也是可以的），此命令删除 `log/` 目录下所有扩展名为 `.log` 的文件以及取消 git 对`log/` 目录下所有扩展名为 `.log` 的文件的跟踪。

##### 移动/重命名文件

在 git 中移动和重命名文件操作是一致的。可以执行：

```javascript
$ git mv <file-from> <file-to>
```

git 会如预期般正常工作，认为这是一次重命名操作。

```javascript
$ git mv README.md ./article/README

$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        renamed:    README.md -> article/README
```

实际上，执行 `git mv README.md ./article/README` 相当于执行了三条命令：

```javascript
$ mv README.md ./article/README
$ git rm README.md
$ git add ./article/README
```

#### 总结

- `git status` 查看当前文件详细状态； `git status -s` 查看简要状态。

- `git add <file-name>` 跟踪暂存文件； `git add .` 跟踪并暂存当前目录层级下所有文件（被忽略的文件除外）。

- `git diff` 查看工作目录文件相对暂存去的修改； `git diff --cached` 或者 `git diff --staged` 查看暂存区文件相对上次提交的修改。

- `git commit` 将暂存区文件提交更新； `git commit -a -m <your-commit>` 跳过使用暂存区提交已跟踪的文件。

- `.gitignore` 忽略文件配置文件。

- `git check-ignore -v <file-name>` 检查文件被哪条规则忽略。

- `git rm <file-name>` 移除文件（磁盘和暂存区）。

- `git rm --cached <file-name>` 仅从暂存区移除。

- `git mv <file-from> <file-to>` 移动/重命名文件。
