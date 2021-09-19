---
title: Git 常用命令汇总
date: 2020-06-06 16:47:48
tags: git
categories: Git
---

> 注：中括号中为可选项

## Git 基础

### 获取 Git 仓库

- `git clone <url> [<local-project-name>]` 从远程服务器克隆仓库至本地目录名下。

- `git init` 初始化本地仓库。

### 远程仓库操作

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

### 记录更新

- `git status` 查看当前文件详细状态； `git status -s` 查看简要状态。

- `git add <file-path>` 跟踪暂存文件； `git add .` 跟踪并暂存当前目录层级下所有文件（被忽略的文件除外）。

- `git diff` 查看工作目录文件相对暂存去的修改； `git diff --cached` 或者 `git diff --staged` 查看暂存区文件相对上次提交的修改。

- `git commit` 将暂存区文件提交更新； `git commit -a -m <your-commit>` 跳过使用暂存区提交已跟踪的文件。

- `.gitignore` 忽略文件配置文件。

- `git check-ignore -v <file-path>` 检查文件被哪条规则忽略。

- `git rm <file-path>` 移除文件（磁盘和暂存区）。

- `git rm --cached <file-path>` 仅从暂存区移除。

- `git mv <file-from> <file-to>` 移动/重命名文件。

### 撤销/重置操作

- `git commit --amend` 会将暂存区中的文件提交，修改提交信息并重新提交（一般用于修正上次提交失误）。

- `git reset [--mixed] <commit-hash> [<file-path>]` 重置提交树/文件。可用选项：

  <details>

  - `--soft` 仅移动 HEAD 指针指向的分支提交，且更改会保留在暂存区，相当于撤销了上一次 `git commit` 命令。

  - `--mixed` （默认选项），不仅会撤销上次提交，还会取消暂存所有更改，相当于回滚到了所有 `git add` 和 `git commit` 命令执行之前。

  - `--hard` （**危险操作**），该命令会撤销提交、`git add` 、 `git commit` 的所有工作并强制覆盖工作区的文件，若被覆盖的文件未提交过则无法恢复。

  - 使用文件路径：如运行 `git reset file.txt` （相当于 `git reset --mixed HEAD file.txt` ），本质上只是将 `file.txt` 从 HEAD 复制到索引（暂存区）中。它还有**取消暂存文件**的实际效果，与 `git add` 的行为正好相反。我们也可以不让 Git 从 HEAD 拉取数据，而是通过具体指定一个提交来拉取该文件的对应版本（如 `git reset eb43bf file.txt` 这样的命令）。

  </details>

- `git checkout -- <file-path>` （**危险操作**） Git 会丢弃你在本地对该文件的任何修改，并且用最近提交的版本覆盖掉它。

> 在 Git 中任何**已提交**的东西几乎总是可以恢复的。然而，任何你未提交的东西丢失后很可能再也找不到了。

### 查看提交历史

`git log` 按时间先后顺序列出所有的提交，最近的更新排在最上面。这个命令会列出每个提交的 SHA-1 校验和、作者的名字和电子邮件地址、提交时间以及提交说明。该命令有许多选项：

#### 限制输出内容

- `-p 或 --patch` 按补丁格式显示每个提交引入的差异。

- `-stat` 显示每次提交的文件修改统计信息（被修改过的文件以及哪些行被移除或是添加了）。

- `--shortstat` 只显示 `--stat` 中最后的行数修改添加移除统计。

- `--name-only` 仅在提交信息后显示已修改的文件清单。

- `--name-status` 仅显示新增、修改、删除的文件清单。

- `--abbrev-commit` 仅显示 SHA-1 校验和的前 7 个字符（共 40 位）。

- `--relative-date` 使用较短的相对时间而不是完整格式显示日期（比如“2 weeks ago”）。

- `--graph` 在日志旁以 ASCII 图形显示分支与合并历史。

- `--color` 使用颜色（可在自定义提交记录中设置）。

  <details>
  <summary>参数选项</summary>

  - `always` 一直可使用颜色。

  - `auto` 如果输出到终端，则使用颜色。

  - `never` 不使用颜色。
  </details>

- `--date` 设置日期格式。

  <details>
  <summary>参数选项</summary>

  - `iso` 以 ISO 8601 格式显示时间戳。

  - `local` 以本地时区显示时间戳。

  - `raw` 以 Git 内置格式显示时间戳（`%s %z`）。

  - `relative` 以相对时间显示（距今多长时间）。

  - `rfc` 以 RFC 2822 格式显示时间戳。

  - `short` 只显示日期。
  </details>

- `--pretty` 使用其他格式显示历史提交信息。可用选项：

  <details>
  <summary>参数选项</summary>

  - `oneline` 将每个提交放在一行显示。

  - `short`、`full`、`fuller` 展示信息的格式与 `git log` 基本一致，只是详尽程度不一。

  - `format` 用来自定义提交记录的显示格式。常用选项：

    - |     选项      |                      说明                       |
      | :-----------: | :---------------------------------------------: |
      |     `%H`      |                提交的完整哈希值                 |
      |     `%h`      |                提交的简要哈希值                 |
      |     `%T`      |                 树的完整哈希值                  |
      |     `%t`      |                 树的简要哈希值                  |
      |     `%P`      |               父提交的完整哈希值                |
      |     `%p`      |               父提交的简要哈希值                |
      |     `%an`     |                    作者名字                     |
      |     `%ae`     |               作者的电子邮件地址                |
      |     `%ad`     | 作者修订日期（可以用 `--date=`选项 来定制格式） |
      |     `%ar`     |          作者修订日期（距今多长时间）           |
      |     `%cn`     |                  提交者的名字                   |
      |     `%ce`     |              提交者的电子邮件地址               |
      |     `%cd`     |                    提交日期                     |
      |     `%cr`     |            提交日期（距今多长时间）             |
      |     `%s`      |                    提交说明                     |
      |     `%d`      |                    分支信息                     |
      | `%C(<color>)` |                    设置颜色                     |
      |   `%Creset`   |                    重置颜色                     |
      |     `%n`      |                      换行                       |

    - > 其中作者指的是实际作出修改的人，提交者指的是最后将此工作成果提交到仓库的人。如：当你为某个项目发布补丁，然后某个核心成员将你的补丁并入项目时，你就是作者，而那个核心成员就是提交者。
    </details>

- `--oneline` `--pretty=oneline --abbrev-commit`的简写。

#### 限制输出长度

- `-<n>` 限制显示的日志条目数量，例如使用 `-2` 选项来只显示最近的两次提交。

#### 常用 log

- `git log --graph --abbrev-commit --color --pretty=format:'%C(bold white)%h%Creset -%C(bold green)%d%Creset %s %C(bold green)(%ar)%Creset %C(bold blue)<%an>%Creset'`

- `git log --graph --abbrev-commit --date=local --color --pretty=format:'%C(bold white)%h%Creset -%C(bold green)%d%Creset %s %C(bold green)(%ad)%Creset %C(bold blue)<%an>%Creset'`

- `git log --graph --abbrev-commit --color --pretty=format:'%C(bold white)%H %d%Creset%n %s %n%C(bold blue)%an <%ae>%Creset %C(bold green)%cr (%ci)'`

### 标签

### 别名

## Git 分支
