# 自定义指令

## 简介

 除了核心功能默认内置的指令(`v-model` 或 `v-show`), Vue 也允许注册自定义指令. 注意，在 Vue 中，代码复用和抽象的主要形式是组件。然而，有的情况下，你仍然需要对普通 DOM 元素进行底层操作，这时候就会用到自定义指令。举个聚焦输入框的例子，如下：

<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="result" data-user="shenqingchuan" data-slug-hash="zYqPwjg" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="自定义指令：基础示例">
  <span>See the Pen <a href="https://codepen.io/shenqingchuan/pen/zYqPwjg">
  自定义指令：基础示例</a> by shenqingchuan (<a href="https://codepen.io/shenqingchuan">@shenqingchuan</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

当页面加载时，该元素将被聚焦 (注意：`autofocus` 在移动版 Safari 上无效)。事实上，只要你在打开这个页面后还没点击过任何内容，这个输入框将会处于聚焦状态。此外，你可以点击 `Rerun` 按钮使输入框处于聚焦状态。

现在让我们创建一个指令来实现这个功能：

```js
const app = Vue.createApp({})
// 注册一个全局自定义指令 `v-focus`
app.directive('focus', {
  // 当被绑定的元素插入到 DOM 中时...
  mounted(el) {
    // 聚焦元素
    el.focus()
  }
})
```
如果想注册局部指令，组件中也接受一个 `directives` 的选项：

```js
directives: {
  focus: {
    // directive definition
    mounted(el) {
      el.focus()
    }
  }
}
```

然后你可以在模板中任何元素上使用新的 `v-focus` attribute，例如：

```html
<input v-focus />
```

## 钩子函数

一个指令定义对象可以提供如下几个钩子函数 (均为可选):

- `beforeMount`: 在指令第一次绑定元素，挂载父组件之前调用。在这里可以进行一次性的初始化设置。

- `mounted`: 在绑定元素的父组件挂载时调用。

- `beforeUpdate`: 在所在组件的 VNode 更新之前调用

:::tip 注意
我们会在[稍后](render-function.html#the-virtual-dom-tree)讨论渲染函数时介绍更多 VNodes 的细节。
:::

- `updated`: 指令所在组件的 VNode 及**其子 VNode** 全部更新后调用。

- `beforeUnmount`: 在卸载绑定元素的父组件之前调用。

- `unmounted`: 仅在指令与元素解除绑定且父组件被卸载时调用一次。

你可以查看这些钩子函数的参数 (即 `el`, `binding`, `vnode`, 和 `prevVnode`) 在 [Custom Directive API](../api/application-api.html#directive)

### 动态指令参数

指令的参数可以是动态的。 例如，在  `v-mydirective:[argument]="value"` 中, `argument` 参数可以根据组件实例数据进行更新！这使得自定义指令可以在应用中被灵活使用。

例如你想要创建一个自定义指令，用来通过固定布局将元素固定在页面上。我们可以像这样创建一个通过指令值来更新竖直位置像素值的自定义指令:

```vue-html
<div id="dynamic-arguments-example" class="demo">
  <p>Scroll down the page</p>
  <p v-pin="200">Stick me 200px from the top of the page</p>
</div>
```

```js
const app = Vue.createApp({})

app.directive('pin', {
  mounted(el, binding) {
    el.style.position = 'fixed'
    // binding.value is the value we pass to directive - in this case, it's 200
    el.style.top = binding.value + 'px'
  }
})

app.mount('#dynamic-arguments-example')
```

这会把该元素固定在距离页面顶部 200 像素的位置。但如果场景是我们需要把元素固定在左侧而不是顶部又该怎么办呢？这时使用动态参数就可以非常方便地根据每个组件实例来进行更新:

```vue-html
<div id="dynamicexample">
  <h3>Scroll down inside this section ↓</h3>
  <p v-pin:[direction]="200">I am pinned onto the page at 200px to the left.</p>
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      direction: 'right'
    }
  }
})

app.directive('pin', {
  mounted(el, binding) {
    el.style.position = 'fixed'
    // binding.arg is an argument we pass to directive
    const s = binding.arg || 'top'
    el.style[s] = binding.value + 'px'
  }
})

app.mount('#dynamic-arguments-example')
```

结果:

<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="result" data-user="shenqingchuan" data-slug-hash="VwarbBm" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="自定义指令：动态参数">
  <span>See the Pen <a href="https://codepen.io/shenqingchuan/pen/VwarbBm">
  自定义指令：动态参数</a> by shenqingchuan (<a href="https://codepen.io/shenqingchuan">@shenqingchuan</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

我们的自定义指令现在已经足够灵活，可以支持一些不同的用例。 为了使它更加动态，我们还可以允许修改一个绑定值。 让我们创建一个额外 property `pinPadding` 并且把它绑定在 `<input type="range">`

```vue-html{4}
<div id="dynamicexample">
  <h2>Scroll down the page</h2>
  <input type="range" min="0" max="500" v-model="pinPadding">
  <p v-pin:[direction]="pinPadding">Stick me 200px from the {{ direction }} of the page</p>
</div>
```

```js{5}
const app = Vue.createApp({
  data() {
    return {
      direction: 'right',
      pinPadding: 200
    }
  }
})
```

现在让我们扩展我们的指令逻辑在组件更新时来重新计算距离:

```js{7-10}
app.directive('pin', {
  mounted(el, binding) {
    el.style.position = 'fixed'
    const s = binding.arg || 'top'
    el.style[s] = binding.value + 'px'
  },
  updated(el, binding) {
    const s = binding.arg || 'top'
    el.style[s] = binding.value + 'px'
  }
})
```

结果:

<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="result" data-user="shenqingchuan" data-slug-hash="PoNOmyO" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="自定义指令：动态参数+动态绑定值">
  <span>See the Pen <a href="https://codepen.io/shenqingchuan/pen/PoNOmyO">
  自定义指令：动态参数+动态绑定值</a> by shenqingchuan (<a href="https://codepen.io/shenqingchuan">@shenqingchuan</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

## 函数简写

在很多时候，你可能想在 `mounted` 和 `updated` 时触发相同行为, 而不关心其它的钩子。你可以通过指令的回调来完成:

```js
app.directive('pin', (el, binding) => {
  el.style.position = 'fixed'
  const s = binding.arg || 'top'
  el.style[s] = binding.value + 'px'
})
```

## 对象字面量

如果指令需要多个值，可以传入一个 JavaScript 对象字面量。记住，指令函数能够接受所有合法的 JavaScript 表达式。

```vue-html
<div v-demo="{ color: 'white', text: 'hello!' }"></div>
```

```js
app.directive('demo', (el, binding) => {
  console.log(binding.value.color) // => "white"
  console.log(binding.value.text) // => "hello!"
})
```

## 组件使用

在 3.0 中，有了 fragments 支持，组件可能有多个根节点。当在具有多个根节点的组件上使用自定义指令时，会产生一个问题。

要解释自定义指令在 3.0 中的组件上的工作细节，我们首先需要了解自定义指令在 3.0 中是如何被编译的。对于这样的指令：

```vue-html
<div v-demo="test"></div>
```

将大致编译成这个:

```js
const vDemo = resolveDirective('demo')

return withDirectives(h('div'), [[vDemo, test]])
```

其中 `vDemo` 将是用户编写的指令对象，它包含 `mounted` 和 `updated` 的钩子。

`withDirectives` 返回一个克隆的 VNode，其中用户钩子被包装并注入为 VNode 生命周期钩子（详情请参阅[渲染函数](render-function.html)）：

```js
{
  onVnodeMounted(vnode) {
    // call vDemo.mounted(...)
  }
}
```

因此，自定义指令会全部包含在 VNode 的 data 中。当在组件上使用一个自定义指令时，这些 `onVnodeXXX` 钩子将会作为外来的 prop 向下传递给组件，同样也在 `this.attrs` 当中。

这也意味着在模板中直接挂钩到元素的生命周期是可能的，当自定义指令太复杂时，这很方便：

```vue-html
<div @vnodeMounted="myHook" />
```

这与[attribute 直落行为](component-attrs.html)是一致的。即组件其他附加 attribute 的规则也同样适用于自定义指令，由子组件决定在何处以及是否应用它。当子组件在内部元素上使用 `v-bind="$attrs"`时，也将会把所有自定义指令应用到该元素上。
