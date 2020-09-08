# 插件

插件是独立的程序，通常用来为 Vue 添加全局功能。可以是包含 `install()` 的 `object`，也可以是个 `function`。

插件没有严格定义的范围，但是插件常见的用途有:

1. 添加全局方法或者 property。如：[vue-custom-element](https://github.com/karol-f/vue-custom-element)。

2. 添加全局资源：指令/过滤器/过渡等。如：[vue-touch](https://github.com/vuejs/vue-touch)

3. 通过全局混入来添加一些组件选项。如：[vue-router](https://github.com/vuejs/vue-router)

4. 通过将一些全局实例方法附加到 `config.globalProperties`.

5. 一个库，提供自己的 API，同时提供上面提到的一个或多个功能。如 [vue-router](https://github.com/vuejs/vue-router)

## 编写插件

为了更好地了解如何创建自己的 Vue.js 插件，我们将创建一个简化版本的显示 `i18n` 字符串插件。

当插件添加到一个应用是，如果它是一个对象，则将调用其 `install` 方法。如果它是一个 `function` ，则调用函数本身。在这两种情况下，它将接收两个参数 -- 由 Vue 的 `createApp` 生成的 `app` 对象，以及用户传递的插件配置项。

让我们从设置插件对象开始。建议在单独的文件中创建并导出，如下所示，保持逻辑的独立性。

```js
// plugins/i18n.js
export default {
  install: (app, options) => {
    // 插件代码
  }
}
```

我们想要一个可用于整个应用的函数来转换多语言 `key`，因此使用 `app.config.globalProperties`。

此函数将接收一个 `key` 字符串，使用翻译字符串在用户提供的插件配置项中查找并返回转换后的字符串。

```js
// plugins/i18n.js
export default {
  install: (app, options) => {
    app.config.globalProperties.$translate = key => {
      return key.split('.').reduce((o, i) => {
        if (o) return o[i]
      }, i18n)
    }
  }
}
```

我们假设用户在使用插件时会传入一个包含 `options` 参数中已翻译字符串的对象。其中 `$translate` 函数将通过诸如 `greetings.hello` 这样的字符串，去用户提供的插件配置项寻找并返回翻译后的值 -- 在本例中是 `你好！`

示例:

```js
greetings: {
  hello: '你好！',
}
```

插件同时也可以使用 `inject` 注入一个 `function` 或者 `attribute` 给插件使用者。比如，应用可以设置 `options` 参数提供翻译对象。

```js
// plugins/i18n.js
export default {
  install: (app, options) => {
    app.config.globalProperties.$translate = key => {
      return key.split('.').reduce((o, i) => {
        if (o) return o[i]
      }, i18n)
    }

    app.provide('i18n', options)
  }
}
```

插件使用者可以在组件中使用 `inject['i18n']` 去获取翻译信息。

此外，由于我们可以访问 `app` 对象中所有其他功能，比如在插件中注入 `mixin` 和 `directive`。了解更多 `createApp` 以及 `app` 实例，请查看文档 [Application API 文档](/api/application-api.html)。

```js
// plugins/i18n.js
export default {
  install: (app, options) => {
    app.config.globalProperties.$translate = (key) => {
      return key.split('.')
        .reduce((o, i) => { if (o) return o[i] }, i18n)
    }

    app.provide('i18n', options)

    app.directive('my-directive', {
      mounted (el, binding, vnode, oldVnode) {
        // 逻辑 ...
      }
      ...
    })

    app.mixin({
      created() {
        // 逻辑 ...
      }
      ...
    })
  }
}
```

## 使用插件

用 `createApp()` 创建 Vue 应用后，你可以使用 `use()` 方法添加插件。

我们将使用在 [编写插件](#编写插件) 章节中创建的 `i18nPlugin` 来进行演示。

`use()` 方法接收 2 个参数。第一个是要安装的插件，在本例中是 `i18nPlugin`。

`use()` 会自动阻止多次注册相同插件，如果重复添加同一个插件 ，默认只会安装一次。

第二个参数是可选的，取决于每个特定的插件本身。在 `i18nPlugin` 的例子中，它是一个带有已翻译字符串的对象。

:::info
如果你正在使用第三方插件，比如 `Vuex` 或 `Vue Router`，那么请查看相关文档，了解这个插件的第二个参数需要接收哪些内容。
:::

```js
import { createApp } from 'vue'
import Root from './App.vue'
import i18nPlugin from './plugins/i18n'

const app = createApp(Root)
const i18nStrings = {
  greetings: {
    hi: 'Hallo!'
  }
}

app.use(i18nPlugin, i18nStrings)
app.mount('#app')
```

了解更多由社区贡献的插件和库请查看 [awesome-vue](https://github.com/vuejs/awesome-vue#components--libraries)。
