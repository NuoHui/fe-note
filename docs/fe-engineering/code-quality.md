## What

### 什么是代码本身的质量?

代码本身的质量: 包括复杂度, 重复率, 代码风格等。

- 复杂度: 项目代码量，模块大小，耦合度等
- 重复率: 重复出现的代码区块占比，通常要求在5%以下(借助平台化工具如Sonar)
- 代码风格: 代码风格是否统一(动态语言代码如JS, Python风格不受约束)


### 代码质量下降恶性循环

常见的代码质量下降的原因:

- 破罐破摔: 在烂代码上迭代代码罪恶感比较小
- 传染性: 不在意代码质量, 只关注业务的产出
- 心有余而力不足

常见的导致恶性循环的场景:

- 业务压力太大

烂代码产生的常见原因是业务压力大，导致没有时间或意愿讲究代码质量。因为向业务压力妥协而生产烂代码之后，开发效率会随之下降，进而导致业务压力更大，形成一种典型的恶性循环。


![](https://user-gold-cdn.xitu.io/2019/9/15/16d32c7f6d759601?w=1244&h=816&f=png&s=172704)

- 通过增加人力解决业务压力

为了应对业务压力，常见的做法就是向项目中增加人力，但是单纯地增加人力的话，会因为风格不一致、沟通成本上升等原因导致烂代码更多。


![](https://user-gold-cdn.xitu.io/2019/9/15/16d32ce9b5c64da1?w=1326&h=510&f=png&s=140669)

那么我们应该如何解决呢?


![](https://user-gold-cdn.xitu.io/2019/9/15/16d32d39b796f928?w=1374&h=672&f=png&s=158241)

这是一个长期坚持的过程。

### 代码质量管控四个阶段

- 规范化

建立代码规范与Code Review制度

1. [airbnb](https://github.com/airbnb/javascript)
2. [standard](https://github.com/standard/standard)
3. [node-style-guide](https://github.com/felixge/node-style-guide)
4. [google javascript style guide](https://google.github.io/styleguide/jsguide.html)
5. [google html/css style guide](https://google.github.io/styleguide/htmlcssguide.html)
6. [Vue风格指南](https://cn.vuejs.org/v2/style-guide/)

我觉得统一项目目录结构也是规范化的一种(比如我们用脚手架创建项目模板), 一个规范化的目录结构大大降低新人的上手成本。

- 自动化

使用工具(linters)自动检查代码质量。


![](https://user-gold-cdn.xitu.io/2019/9/15/16d32d46d3e89065?w=1254&h=1122&f=png&s=224538)

- 流程化

将代码质量检查与代码流动过程绑定。


![](https://user-gold-cdn.xitu.io/2019/9/15/16d32d4edac16516?w=1644&h=236&f=png&s=87006)

质量检查与代码流动绑定后的效果：


![](https://user-gold-cdn.xitu.io/2019/9/15/16d32d53cdbc31c9?w=1666&h=364&f=png&s=140003)


```
备注:

1. 编辑时候: 通过编辑器插件, 实时查看质量检查

2. 可以利用CI(Jekins/Travis)把构建发布过程搬到线上, 先跑代码扫描, 测试代码等, 然后没有错误再进行build, build成功通过ssh推到服务器。
```

- 中心化

以团队整体为视角，集中管理代码规范，并实现质量状况透明化。

当团队规模越来越大，项目越来越多时，代码质量管控就会面临以下问题：

1. 不同项目使用的代码规范不一样

2. 部分项目由于放松要求，没有接入质量检查，或者存在大量未修复的缺陷

3. 无法从团队整体层面上体现各个项目的质量状况对比

为了应对以上问题，需要建设中心化的代码质量管控体系，要点包括：

代码规范统一管理。通过脚手架命令垂直管理代码扫描配置规则集, 自动安装，不在本地写规则。一个团队、一类项目、一套规则。

* * *


* [待定] <u>使用统一的持续集成服务(Jekins/Travis等)。质量检查不通过的项目不能上线。</u>

* [待定]<u> 建立代码质量评分制度(借助Sonar)。让项目与项目之间能够横向对比，项目自身能够纵向对比，并且进行汇总反馈。</u>

## Why

> 代码质量是团队技术水平和管理水平的直接体现。

> 看代码的时间远远多于写代码的时间

### 目前前端项目出现的问题

- 书写风格不统一, 阅读体验差
- 维护性差, 复用性差(Code Review互相进步)
- 容易出现低质量代码, 代码返工率高
- git commit不规范


## How

通过哪些手段来保证代码质量?

### EditorConfig

[EditorConfig]( https://editorconfig.org/)在多人协作开发项目时候, 支持跨编辑器, IDE来支持维护一致的编码样式(文件格式)。

VSCode插件EditorConfig for VS Code提供一键生成.editorconfig。

查看[实例](https://editorconfig.org/#example-file)。

### TypeScript

- [官网介绍](https://www.typescriptlang.org/
)。
- [中文awesome-typescript](https://github.com/semlinker/awesome-typescript)
- [TypeScript体系调研报告](https://juejin.im/post/59c46bc86fb9a00a4636f939)
- [2018年度JS趋势报告](https://2018.stateofjs.com/javascript-flavors/overview/)

### Git Hooks

Git能在特定的重要动作发生时触发自定义脚本。 有两组这样的钩子：客户端的和服务器端的。 客户端钩子由诸如提交和合并这样的操作所调用，而服务器端钩子作用于诸如接收被推送的提交这样的联网操作, 我们目前使用的大多数是客户端钩子。

通过[husky](https://github.com/typicode/husky)集成[git hooks](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90), 如果对git想有更全面的理解推荐阅读[GIt文档](https://git-scm.com/book/zh/v2)。

husky会安装一系列的git hook到项目的.git/hook目录中。

下面两张图分别对比没有安装husky与安装了husky的git目录区别:


![](https://user-gold-cdn.xitu.io/2019/9/15/16d32d74c11d9f41?w=1314&h=688&f=png&s=332033)

![](https://user-gold-cdn.xitu.io/2019/9/15/16d32d7d8776a6d8?w=1286&h=1100&f=png&s=510934)

当你用 git init 初始化一个新版本库时，Git 默认会在这个目录中放置一些示例脚本(.sample结尾的文件)。

#### pre-commit


pre-commit 钩子在键入提交信息前运行。 它用于检查即将提交的快照，你可以利用该钩子，来检查代码风格是否一致（运行类似 lint 的程序。


- [lint-staged](https://github.com/okonet/lint-staged): 可以获取所有被提交的文件并执行配置好的任务命令,各种lint校验工具可以配置好lint-staged任务中。
- [prettier](https://prettier.io/): 可以配置到lint-staged中, 实现自动格式化编码风格。
- [stylelint](https://github.com/stylelint/stylelint)
- [eslint](https://cn.eslint.org/)
- [tslint](https://github.com/palantir/tslint)
- [eslint-plugin-vue](https://github.com/vuejs/eslint-plugin-vue): Vue.js官方推荐的lint工具


关于[为什么选择prettier, 以及eslint 与prettier区别?](https://zhuanlan.zhihu.com/p/62542268)。

关于[prettier配置](https://prettier.io/docs/en/configuration.html)。
关于[stylelint配置](https://stylelint.io/user-guide/configuration/)。
关于[eslint配置](https://cn.eslint.org/docs/user-guide/configuring)。

#### commit-msg


- [commitlint](https://github.com/conventional-changelog/commitlint)。


commit-msg 可以用来在提交通过前验证项目状态或提交信息, 使用该钩子来核对提交信息是否遵循指定的模板。

关于git hooks在package.json配置:


![](https://user-gold-cdn.xitu.io/2019/9/15/16d32da5cdaf3df6?w=754&h=892&f=png&s=181244)

### 测试

#### unittest

- [Jest](https://jestjs.io/
)
- [Mocha](https://mochajs.org/
)

#### e2e

- [Nightwatch](http://nightwatchjs.org/
)
- [Cypress](https://www.cypress.io/
)


### CHANGELOG

更新日志, [standard-version](https://github.com/conventional-changelog/standard-version)。

### Code Review

* [待定] Review制度,我们目前公司在代码merge时候多人审核才通过。


## 如何快速落地到当前业务


### 前端脚手架(xx-cli)

采用中心化集中管理代码扫描配置文件的思路, 把code lint配置文件做成一个npm包发到内网, 然后扩展脚手架命令一键执行下发远程配置文件到本地项目, 并且把新增的package.json依赖打进来, 大家后面再安装新的依赖即可。

所谓中心化管理: 所有项目代码配置文件以远程配置文件为准, 如果你本地有同名配置会被删除, 这样方便后续我们更新配置文件比如(增加vw/vh适配), 以及所有业务同步问题。

```
目前只有基于vue.js项目的lint脚本命令, 后续有别的项目, 考虑通过

dc-cli lint -- vue
dc-cli lint -- node
扩展
```


### demo演示

demo演示如何在旧项目中植入代码质量检测?
由于这部分是在内网演示就不发不出来了。

至于脚手架可以参考我之前的demo[easy-cli](https://github.com/NuoHui/easy-cli)。这是比较全的demo。


## Future

### Jekins自动化


### [Sonar](https://www.sonarqube.org/)

[Github:](https://github.com/SonarSource/sonarqube)

SonarQube 是一款领先的持续代码质量监控平台，开源在github 上，可以轻松配置在内网服务器，实时监控代码，帮助了解提升提升团队项目代码质量。通过插件机制，SonarQube可以继承不同的测试工具，代码分析工具，以及持续集成工具。

与持续集成工具（例如 Hudson/Jenkins 等）不同，SonarQube 并不是简单地把不同的代码检查工具结果（例如 FindBugs，PMD 等）直接显示在 Web 页面上，而是通过不同的插件对这些结果进行再加工处理，通过量化的方式度量代码质量的变化，从而可以方便地对不同规模和种类的工程进行代码质量管理。

行业内提到"代码质量管理, 自动化质量管理", 一般指的都是通过Sonar来实现。

用Sonar能够实现什么?
- 技术债务(sonar根据"规则"扫描出不符合规则的代码)
- 覆盖率(单元测试覆盖率)
- 重复(重复的代码, 有利于提醒封装)
- 结构
- …

### sonarjs

sonar支持多种编程语言, 其中包括JavaScript. 如[sonarjs](https://www.sonarsource.com/products/codeanalyzers/sonarjs.html)


## 最后打个广告,欢迎关注我的公众号 全栈小窝。
