# cn-v3.vuejs.org

目前本中文 Vue.js 3.0 文档仓库由 [@ShenQingchuan](https://github.com/ShenQingchuan) 个人进行维护，站点目前部署于 [个人腾讯云站点 vue.techdict.pro](https://vue.techdict.pro)。

本站由 [VuePress](https://vuepress.vuejs.org/) 搭建而成，站点内文章皆由 Markdown 格式书写，存放于 `src` 文件夹中。

## 编写

请查阅 [Vue 文档编写指南](https://v3.vuejs.org/guide/writing-guide.html) 了解维护文档的约定规则和推荐。

> 官方英文文档还在测试阶段：目前并不建议大规模的团队合作进行翻译工作，所有内容可能随时会更改。如果您看到一个问题，希望引起我们的注意，请[创建一个 issue](https://github.com/Shenqingchuan/docs-next/issues/new)，我们会及时处理。

## 开发

1. 克隆本仓库

```bash
git clone https://github.com/ShenQingchuan/docs-next.git
```

2. 安装依赖

```bash
yarn # or npm install
```

3. 启动本地开发环境

```bash
yarn serve # or npm run serve
```

## 翻译合作

已为本仓库创建好了 Jenkins 自动化构建流程。若您对翻译有任何贡献的想法，请自行开启一个新的分支，翻译其中的某篇文档，然后向这里提交 Pull Request。

### 翻译要求

由于 docs-next 官方英文仓库采用了大量的 codepen.io 的 demo 展示，有很多内置示例的内容也存在大量英文，**它们也应当是文档的一部分！**如果您在翻译某篇文档时遇到 codeopen.io 的 embed demo，**请不要改动他们**，而是在 Pull Request 当中留下说明。

目前我已经 fork 了官方团队的部分 demo，并将其中必要的英文部分完成了翻译替换。为了保证之后访问的统一性，暂时采取这样的合作模式。

### 避免冲突

在翻译之前，您最好首先添加官方英文文档仓库的 git origin，命名为 `official`，然后在开始工作之前，执行 `git pull official master` 来查看最新的英文版本的动向。

**请注意，官方英文文档采用了 algolia 但本仓库当中暂时移除了对该模块及其他本项目自建组件的使用。**（并为删除，只是暂时注释掉了）

然后在编辑器（如 VSCode）当中全局查看该文档目前的 TODO，避免先去翻译该部分的文档，因为有可能有较大变动。

首先最紧要的任务是，将 [Hexo 版的中文站点](https://cn.vuejs.org) 上的内容和新版文档内容进行对比、更新。
