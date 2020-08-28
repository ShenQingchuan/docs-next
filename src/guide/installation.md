# 安装

## 发行说明

最近一次 beta 版本: 3.0.0-rc.5

每个版本的详细发布说明可以在 [GitHub](https://github.com/vuejs/vue-next/releases) 上找到。

## Vue Devtools

> 目前在测试版本
>
> Vue 3 版本相应的 Vue Devtools 至少需要 `vue@^3.0.0-rc.1`

在使用 Vue 时，我们推荐在你的浏览器上安装 [Vue Devtools](https://github.com/vuejs/vue-devtools#vue-devtools)。它允许你在一个更友好的界面中审查和调试 Vue 应用。

[获取 Chrome 扩展](https://chrome.google.com/webstore/detail/vuejs-devtools/ljjemllljcmogpfapbkkighbhhppjdbg)

[获取 Firefox 插件](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)

[获取独立 Electron 应用](https://github.com/vuejs/vue-devtools/blob/dev/packages/shell-electron/README.md)

## CDN

对于制作原型或学习，你可以这样使用最新版本：

```html
<script src="https://unpkg.com/vue@next"></script>
```

对于生产环境，我们推荐链接到一个明确的版本号和构建文件，以避免新版本造成的不可预期的破坏：

## NPM

在用 Vue 构建大型应用时推荐使用 NPM 安装。NPM 能很好地和诸如 [Webpack](https://webpack.js.org/) 或 [Browserify](http://browserify.org/) 模块打包器配合使用。同时 Vue 也提供配套工具来开发[单文件组件](../guide/single-file-component.html)。

```bash
# 最新稳定版
$ npm install vue@next
```

## CLI

Vue 提供了一个[官方的 CLI](https://github.com/vuejs/vue-cli)，为单页面应用 (SPA) 快速搭建繁杂的脚手架。它为现代前端工作流提供了 batteries-included 的构建设置。只需要几分钟的时间就可以运行起来并带有热重载、保存时 lint 校验，以及生产环境可用的构建版本。更多详情可查阅 [Vue CLI 的文档](https://cli.vuejs.org)。

::: tip 注意
CLI 工具假定用户对 Node.js 和相关构建工具有一定程度的了解。如果你是新手，我们强烈建议先在不用构建工具的情况下通读<a href="./">指南</a>，在熟悉 Vue 本身之后再使用 CLI。
:::

对于 Vue 3，至少需要使用 Vue CLI v4.5 以上版本，在 `npm` 上发布为 `@vue/cli@next`。若要升级，你需要重新全局安装最新版本的 `@vue/cli`。

```bash
yarn global add @vue/cli@next
# 或
npm install -g @vue/cli@next
```

在 Vue 项目当中，运行：

```bash
vue upgrade --next
```

## Vite

[Vite](https://github.com/vitejs/vite) 是一种 Web 开发构建工具，通过其原生的 ES 模块导入方法提供高效、快速的服务。

可以通过在终端中输入以下命令，使用 Vite 快速启动 Vue 项目：

通过 NPM：

```bash
npm init vite-app <project-name>
cd <project-name>
npm install
npm run dev
```

或者通过 Yarn：

```bash
yarn create vite-app <project-name>
cd <project-name>
yarn
yarn dev
```

## 对不同构建版本对解释

在 [NPM 包的 `dist/` 目录下](https://cdn.jsdelivr.net/npm/vue@3.0.0-rc.1/dist/)你可以找到 Vue.js 的不同构建版本。这里列出了它们之间的差别：

## 术语

- **完整版**：同时包含编译器和运行时的版本。

- **编译器**：用来将模板字符串编译成为 JavaScript 渲染函数的代码。

- **运行时**：用来创建 Vue 实例、渲染并处理虚拟 DOM 等的代码。基本上就是除去编译器的其它一切。

### 使用 CDN 或不使用任何打包工具

#### `vue(.runtime).global(.prod).js`

- 对于在浏览器中直接使用 `<script src="...">` 的情况，会全局暴露 Vue 对象。
- 浏览器中的模板模板编译：
  - `vue.global.js` 为完整版，同时包含了编译器与运行时，所以它支持随时编译解析模板。
  - `vue.runtime.global.js` 仅包含运行时，因此需要模板已经在一个构建步骤中被预先编译好。
- 内联引入所有 Vue 内部核心的包 - 即一个没有任何其他依赖的单文件。这意味着你需要从这个文件中引入所有内容，并且仅是为了确保获得相同的代码实例而导入该文件。
- 包含硬编码的 prod / dev 分支，并且 prod 构建时已经预先最小化压缩。并在生产环境中使用这些 `*.prod.js` 文件。

:::tip 注意
全局构建版本并不是 [UMD](https://github.com/umdjs/umd) 构建版本。而是构建为了[立即执行表达式 IIFEs](https://developer.mozilla.org/en-US/docs/Glossary/IIFE) ，也只是为了能在以 `<script src="...">` 导入的场景下直接使用。
:::

#### `vue(.runtime).esm-browser(.prod).js`

- 通过原生 ES 模块的方式导入使用。（在浏览器中通过 `<script type="module">`）
- 与全局构建版本共享相同的运行时编译、依赖内联和硬编码的 prod / dev 行为。

### 使用打包工具

#### `vue(.runtime).esm-bundler.js`

- 适于结合如 `webpack`, `rollup` and `parcel` 之类的打包工具使用。
- 通过 `process.env.NODE_ENV` 来区分 prod / dev 分支的行为（必须由打包工具替换）
- 未最小化压缩（随其他代码在打包后完成）
- 引入了相应依赖 (如： `@vue/runtime-core`, `@vue/runtime-compiler`)
  - 引入的依赖同样是 esm-bundler 形式的构建版本，并且同样包含了其自身的依赖 (如： `@vue/runtime-core` 引入了 `@vue/reactivity`)
  - 这意味着你**可以**单独地安装/引入这些依赖，无需使用不同依赖的模块名来结尾，但你必须确保它们都为相同版本.
- 浏览器中的模板编译:
  - `vue.runtime.esm-bundler.js`：**默认** 只包含运行时，需要所有模板被预先编译好。这将会是打包工具的入口文件。(通过 `package.json` 中的 `module` 字段) 因为当使用一个打包工具时，模板通常会被预先编译完成。
  - `vue.esm-bundler.js`：包含运行时的编译器。若你使用打包工具但仍然想要运行时的模板编译（如：DOM 上使用模板功能或是 JavaScript 字符串形式的模板)。你将需要配置你的打包工具，将这个文件与 Vue 关联。

### 对于服务端渲染

- `vue.cjs(.prod).js`:
  - 对于在 Node.js 服务端渲染请使用 `require()`。
  - 如果你使用 webpack 打包，设置了 `target: 'node'` 目标进行打包并已经正确外部化 `vue`，你应当使用这个构建版本。
  - dev/prod 两个分支的文件已经预编译好，而具体是哪个取决于 `process.env.NODE_ENV`。

## 运行时 + 编译器 vs. 仅含运行时

如果你需要在客户端编译模板 (比如传入一个字符串给 template 选项，或挂载到一个元素上并以其 DOM 内部的 HTML 作为模板)，就将需要加上编译器，即完整版：

```js
// 需要编译器
Vue.createApp({
  template: '<div>{{ hi }}</div>'
})

// 不需要编译器
Vue.createApp({
  render() {
    return Vue.h('div', {}, this.hi)
  }
})
```

当使用 vue-loader 或 vueify 的时候，\*.vue 文件内部的模板会在构建时预编译成 JavaScript。你在最终打好的包里实际上是不需要编译器的，所以只用运行时版本即可。
