# 模板 ref

> 该页面假设你已经阅读过了[组件基础](component-basics.md)。如果你还对组件不太了解，推荐你先阅读它。

除了 props 和事件，有时你可能仍然需要在 JavaScript 中直接访问一个子组件。要实现这一点，您可以使用 `ref` attribute 为子组件或 HTML 元素分配一个引用 ID。例如：

```html
<input ref="input" />
```

这将十分有用的，例如，在组件挂载完成后，自动聚焦于该 input 元素。

```js
const app = Vue.createApp({})

app.component('base-input', {
  template: `
    <input ref="input" />
  `,
  methods: {
    focusInput() {
      this.$refs.input.focus()
    }
  },
  mounted() {
    this.focusInput()
  }
})
```

或者你也可以给这个组件自身加上一个 `ref` 然后在父组件当中使用它来触发 `focusInput` 事件。

```html
<base-input ref="usernameInput"></base-input>
```

```js
this.$refs.usernameInput.focusInput()
```

当 `ref` 和 `v-for` 一同使用时，获得的 ref 将是一个数组，其中包含镜像数据源的子组件。

::: warning 警告
`$refs` 只能在组件被渲染完成后使用。它只是提供了一种能够直接操纵子元素的方式。——应当避免在模板和计算属性当中使用 `$refs`。
:::

**了解更多，请查看**: [在组合式 API 中使用模板 ref](/guide/composition-api-template-refs.html#template-refs)
