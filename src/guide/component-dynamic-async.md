# 动态 & 异步组件

> 该页面假设你已经阅读过了[组件基础](component-basics.md)。如果你还对组件不太了解，推荐你先阅读它。

## 在动态组件上使用 `keep-alive`

我们之前曾经在一个多标签的界面中使用 is attribute 来切换不同的组件：

```vue-html
<component :is="currentTabComponent"></component>
```

当在这些组件之间切换的时候，你有时会想保持这些组件的状态，以避免反复重渲染导致的性能问题。例如我们来展开这个多标签界面：

<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="result" data-user="shenqingchuan" data-slug-hash="YzqVVdd" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="动态组件：没有 keep-alive">
  <span>See the Pen <a href="https://codepen.io/shenqingchuan/pen/YzqVVdd">
  动态组件：没有 keep-alive</a> by shenqingchuan (<a href="https://codepen.io/shenqingchuan">@shenqingchuan</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

你会注意到，如果你选择了一篇文章，切换到 Archive 标签，然后再切换回 Posts，是不会继续展示你之前选择的文章的。这是因为你每次切换新标签的时候，Vue 都创建了一个新的 `currentTabComponent` 实例。

重新创建动态组件的行为通常是非常有用的，但是在这个案例中，我们更希望那些标签的组件实例能够被在它们第一次被创建的时候缓存下来。为了解决这个问题，我们可以用一个 `<keep-alive>` 元素将其动态组件包裹起来。

```html
<!-- 未激活的组件将会被缓存！-->
<keep-alive>
  <component :is="currentTabComponent"></component>
</keep-alive>
```

来看看修改后的结果：

<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="result" data-user="shenqingchuan" data-slug-hash="RwaVgNz" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="动态组件：没有 keep-alive">
  <span>See the Pen <a href="https://codepen.io/shenqingchuan/pen/RwaVgNz">
  动态组件：没有 keep-alive</a> by shenqingchuan (<a href="https://codepen.io/shenqingchuan">@shenqingchuan</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

现在这个 _诗歌_ 标签保持了它的状态 (被选中的诗歌) 甚至当它未被渲染时也是如此。你可以在这个示例查阅到完整的代码。

:::danger 注意

注意这个 `<keep-alive>` 要求被切换到的组件都有自己的名字，不论是通过组件的 name 选项还是局部/全局注册。

:::

你可以在 [API 参考文档](../api/built-in-components.html#keep-alive)查阅更多关于 `<keep-alive>` 的细节。

## 异步组件

<common-watch-video title="Watch a free video lesson on Vue School" href="https://vueschool.io/lessons/dynamically-load-components?friend=vuejs" />

在大型应用中，我们可能需要将应用分割成小一些的代码块，并且只在需要的时候才从服务器加载一个模块。为了使其可行，Vue 提供了一个 `defineAsyncComponent` 方法：

```js
const app = Vue.createApp({})

const AsyncComp = Vue.defineAsyncComponent(
  () =>
    new Promise((resolve, reject) => {
      resolve({
        template: '<div>I am async!</div>'
      })
    })
)

app.component('async-example', AsyncComp)
```

如你所见，该方法接收一个返回 `Promise` 的工厂函数，该 Proimse 的 `resolve` 回调会在你从服务器得到组件定义的时候被调用，你也可以调用 `reject(reason)` 来表示加载失败。

由于可以在工厂函数中返回一个 `Promise`，所以在 Webpack 2 或更高版本与 ES2015 语法中你可以这样做：

```js
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent(() =>
  import('./components/AsyncComponent.vue')
)

app.component('async-component', AsyncComp)
```

当[局部注册一个组件时](component-registration.html#local-registration)，你也可以使用 `defineAsyncComponent`：

```js
import { createApp, defineAsyncComponent } from 'vue'

createApp({
  // ...
  components: {
    AsyncComponent: defineAsyncComponent(() =>
      import('./components/AsyncComponent.vue')
    )
  }
})
```

### 与 Suspense 一起使用

异步组件在默认情况下是可挂起的。这意味着如果它在父链中有一个 `<Suspense>`，它将被视为该 `<Suspense>` 的异步依赖。在这种情况下，加载状态将由 `<Suspense>` 控制，组件自身的加载、错误、延迟和超时选项将被忽略。

异步组件可以选择退出 `Suspense` 控制，并通过在其选项中指定 `suspensable:false`，让组件始终控制自己的加载状态。

你可以在中查看可用选项的列表 [API 参考](../api/global-api.html#arguments-4)
