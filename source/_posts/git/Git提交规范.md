# 使用 commitizen 规范 git 提交

## 为什么要规范 git 提交

git 每次的代码提交，都需要写一份提交说明，否则不允许提交。

```shell
git commit -m <commit message>
```

上面命令可快速编写并提交一行提交说明。

如果一行说明不够，可执行 `git commit` ，会启动指定的文本编辑器，允许你写多行说明。

git 本身的提交命令并不会限制提交内容，但是规范的 commit message 还是有诸多好处的：

- 提供更多的修改信息，方便快速浏览。
- 可以通过 `--grep` 选项过滤提交，便于快速查找信息。
- 可以直接从 commit 生成 Change log（发布新版本时用于说明版本差异的文档）。

## commit message 格式

commit message 包括三个部分：Header、Body 和 Footer：

```shell
<type>(<scope>): <subject>

<body>

<footer>
```

其中 Header 是必需的，Body 和 Footer 可省略。

不管是哪一个部分，任何一行都不得超过 100 个字符。这是为了避免自动换行影响美观。

### Header

Header 部分只有一行，包括三个字段：type（必需）、scope（可选）和 subject（必需）。

#### type

type 用于说明 commit 类别，一般只允许以下几个标识：

- `feat`: 新功能。
- `fix`: 修复 bug。
- `docs`: 针对文档的修改。
- `style`: 代码格式修改（不影响代码运行的变动）。
- `refactor`: 代码重构（不增加新功能，也不修复 bug）。
- `perf`: 性能优化。
- `test`: 修改测试用例。
- `build`: 影响构建流程或外部依赖的变动（npm, webpack）。
- `ci`: 修改 CI 配置文件或脚本。
- `chore`: 其他不影响程序功能和测试用例的变动。
- `revert`: 回滚提交（建议附上回滚 commit 的 hash 及 Header）。

#### scope

scope 用于说明本次 commit 的影响范围，具体内容由项目决定。

#### subject

subject 为本次 commit 的简要说明，一般不超过 70 字。建议使用祈使句。

### Body

Body 是对本次 commit 改动的详细描述，可以写成多行。建议使用祈使句。

### Footer

Footer 部分一般只用于两种情况：

#### 不兼容变动

如果代码改动与上一个版本不兼容，则 Footer 部分需要以`BREAKING CHANGE`开头，后面是对变动的描述、以及变动理由和迁移方法。

#### 关闭 Issue

如果本次 commit 会影响某个 issue，可以在 Footer 部分附加该 issue 信息，如`close #233` 、`fix #233`、`re #233`。

## 使用 commitzen

如果每次提交都需要纯手动输入提交说明，难免会觉得些许麻烦。我们可以使用`commitzen`工具来帮助生成符合格式的 commit message。

### 安装

#### 全局安装

```shell
npm install -g commitizen
```

现在可以使用 `git cz` 代替 `git commit` 进行提交。

#### 在项目中安装

如果你使用的`npm`版本大于**5.2**，可以局部安装并使用`npx cz` 以取代全局安装，或者写入 npm script 中。

如果你的项目是 commitzen 友好的，那系统将按项目制定的格式进行提示并格式化 commit message。否则，可使用`cz-conventional-changelog`插件快速设置。

### 使用 cz-conventional-changelog

#### 全局安装

```shell
npm install -g cz-conventional-changelog
```

commitzen 的全局配置文件位于`~/.czrc`，执行：

```shell
echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc
```

> Windows 环境下，执行 `git cz` 时可能会报错：
>
> ```shell
> The config file at "C:\Users\Administrator\.czrc" contains invalid charset, expect utf8
> ```
>
> 在 PowerShell 中 echo 生成的`.czrc`为`UTF-16`，只需重新手动创建该文件并写入内容即可。

#### 在项目中安装

```shell
commitizen init cz-conventional-changelog --save-dev --save-exact
```

如果使用`yarn`：

```shell
commitizen init cz-conventional-changelog --yarn --dev --exact
```

然后在**package.json**文件中配置：

```json
...
  "config": {
    "commitizen": {
      "path": "cz-conventional-changelog"
    }
  }
...
```

或者添加一个`.czrc` 或 `.cz.json` 文件：

```json
{
  "path": "cz-conventional-changelog"
}
```

现在你可以使用`git cz` 替代`git commit`的所有操作，比如：`git cz -a`。
