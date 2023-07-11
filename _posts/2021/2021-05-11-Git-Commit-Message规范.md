---

title:  "Git-Commit-Message规范"
date:   2021-05-11 14:29:36 +0800
categories: git
typora-root-url: ..
---
# Git-Commit-Message规范

Git 每次提交代码，都要写 Commit message（提交说明），否则就不允许提交。

```
$ git commit -m "hello world"
```

上面代码的`-m`参数，就是用来指定 commit mesage 的。

如果一行不够，可以只执行`git commit`，就会跳出文本编译器，让你写多行。

```
$ git commit
```

基本上，写什么都行,  但还是有多种 Commit message 的[写法规范](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fajoslin%2Fconventional-changelog%2Fblob%2Fmaster%2Fconventions)。本文介绍Angular 规范，这是目前使用最广的写法，比较合理和系统化，并且有配套的工具。

## 1 Commit Message 的格式

每次提交，Commit message 都包括三个部分：Header（type, scope, subject），Body 和 Footer。

```
 		<type>(<scope>): <subject>
        <BLANK LINE>
        <body>
        <BLANK LINE>
        <footer>
```

其中，Header 是必需的，Body 和 Footer 可以省略。

不管是哪一个部分，任何一行都不得超过72个字符（或100个字符）。这是为了避免自动换行影响美观。

### 1.1 Header

Header部分只有一行，包括三个字段：`type`（必需）、`scope`（可选）和`subject`（必需）。

**（1）type**

`type`用于说明 commit 的类别，只允许使用下面7个标识。

- feat：新功能（feature）
- fix：修补bug
- docs：文档（documentation）
- style： 格式（不影响代码运行的变动）
- refactor：重构（即不是新增功能，也不是修改bug的代码变动）
- perf: 性能改善
- test：增加测试
- build：编译和外部依赖的变动
- ci: 持续集成配置变动
- chore：构建过程或辅助工具的变动

**（2）scope**

`scope`用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，小写。表示变更的包或模块范围，可多个组合，若涉及范围较大，可用 * 代替。各服务可以自行定义，组内同学可轻易理解。通用scope列表如下：

- dto: dto结构变化
- core: core包
- service: service层代码
- dao: dao层代码
- sql: sql代码变更

除上述通用字段外，Scope中各方向可自行定义关键字.

**（3）subject**

`subject`是 commit 目的的简短描述，不超过50个字符。

- 以动词开头，使用第一人称现在时，比如`change`，而不是`changed`或`changes`
- 第一个字母小写
- 结尾不加句号（`.`）

### 1.2 Body

Body 部分是对本次 commit 的详细描述，可以分成多行。下面是一个范例。

```
More detailed explanatory text, if necessary.  Wrap it to 
about 72 characters or so. Further paragraphs come after blank lines.- Bullet points are okay, too- Use a hanging indent
```

有两个注意点。

（1）使用第一人称现在时，比如使用`change`而不是`changed`或`changes`。

（2）应该说明代码变动的动机，以及与以前行为的对比。

### 1.3 Footer

Footer 部分只用于两种情况。

**（1）不兼容变动**

如果当前代码与上一个版本不兼容，则 Footer 部分以`BREAKING CHANGE`开头，后面是对变动的描述、以及变动理由和迁移方法。

```
BREAKING CHANGE: isolate scope bindings definition has changed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }

    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

**（2）关闭 Issue**

如果当前 commit 针对某个issue，那么可以在 Footer 部分关闭这个 issue 。

```
Closes #234
```

也可以一次关闭多个 issue 。

```
Closes #123, #245, #992
```

### 1.4 Revert

还有一种特殊情况，如果当前 commit 用于撤销以前的 commit，则必须以`revert:`开头，后面跟着被撤销 Commit 的 Header。

```
revert: feat(pencil): add 'graphiteWidth' option

This reverts commit 667ecc1654a317a13331b17617d973392f415f02.
```

Body部分的格式是固定的，必须写成`This reverts commit <hash>.`，其中的`hash`是被撤销 commit 的 SHA 标识符。

如果当前 commit 与被撤销的 commit，在同一个发布（release）里面，那么它们都不会出现在 Change log 里面。如果两者在不同的发布，那么当前 commit，会出现在 Change log 的`Reverts`小标题下面。



## 2 配置IDEA插件检查commit message

### 1) GitToolBox

![img](/assets/images\GitToolBox.png)



主要功能：在Idea的状态栏显示git状态，还提供了定时fecth等功能。

可以在Settings|Other Settings 配置commit message validation 

```
(feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(\(.*\)): (.|\n)*
```

![GitToolBox Porject](/assets/images\git-tool-box.png)

**Features**

- Git status display

  - shows number of ahead / behind commits for current branch as status bar widget
  - ahead / behind, current branch, tags on HEAD as Project View decoration on modules
  - can use remote tracking branch or any other 'parent' branch

- Status bar widget

  git status

  - tooltip shows info for all repositories
  - popup menu - status refresh
  - popup menu - repository fetch
    **git blame**
  - tooltip shows detailed blame for file/line
  - popup - detailed blame balloon
  - action to open detailed blame balloon

- **Auto fetch** - runs git fetch at fixed intervals

- **Push tags on current branch** - available in VCS / Git

- **Behind tracker** - shows notification when behind count of current branch changes and is non-zero

- **Branch name completion in Commit dialog** - provides branch name completion inside Commit dialog message

- **Blame inlined in active editor** - show blame for line at caret in active editor

- **Git Extender integration** - can be selected as update action executed from behind tracker popup

### 2) Git Commit Template
在提交commit，可以使用模板创建message信息:

![git commit template](/assets/images\git-commit-template.png)

两个插件配合使用效果：

![](/assets/images\Git-commit-message.png)

