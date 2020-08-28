# 自定义事件

> 该页面假设你已经阅读过了[组件基础](component-basics.md)。如果你还对组件不太了解，推荐你先阅读它。

## 事件名

不同于组件和 prop，事件名不存在任何自动化的大小写转换。而是触发的事件名需要完全匹配监听这个事件所用的名称。举个例子，如果触发一个 camelCase 名字的事件：

```js
this.$emit('myEvent')
```

则监听这个名字的 kebab-case 版本是不会有任何效果的：

```html
<!-- 没有效果 -->
<my-component @my-event="doSomething"></my-component>
```

不同于组件和 prop，事件名不会被用作一个 JavaScript 变量名或 property 名，所以就没有理由使用 camelCase 或 PascalCase 了。并且 `v-on` 事件监听器在 DOM 模板中会被自动转换为全小写 (因为 HTML 是大小写不敏感的)，所以 `@myEvent` 将会变成 `@myevent`——导致 `myEvent` 不可能被监听到。

因此，我们推荐你**始终使用 kebab-case 的事件名**。

## 自定义组件的 v-model

可以通过 `emit` 选项在组件上定义所发出的事件。

```js
app.component('custom-form', {
  emits: ['in-focus', 'submit']
})
```

当原生事件（例如 `click` ）在 `emit` 选项中定义时，它将被组件中的事件覆盖，而不会调用其相应的原生事件侦听器。

::: tip 注意
建议定义出所有会触发的事件，以便更好地记录组件应该如何工作。
:::

### 验证触发的事件

与 prop 类型验证类似，如果是使用对象语法而不是数组语法定义触发的事件，则可以对其进行验证。

要添加验证，将为事件分配一个函数，该函数接收传递给 `$emit` 调用的参数，并返回一个布尔值来表示事件是否有效。

```js
app.component('custom-form', {
  emits: {
    // 没有验证
    click: null,

    // 验证 submit 事件
    submit: ({ email, password }) => {
      if (email && password) {
        return true
      } else {
        console.warn('非法的 submit 事件 payload！')
        return false
      }
    }
  },
  methods: {
    submitForm() {
      this.$emit('submit', { email, password })
    }
  }
})
```

## `v-model` 参数

默认地，一个组件上的 `v-model` 使用 `modelValue` 作为 prop，以 `update:modelValue` 为事件。我们可以通过给 `v-model` 传递一个参数来提交修改这些名字：

```html
<my-component v-model:foo="bar"></my-component>
```

在此例中，子组件会期望一个 `foo` prop 和触发 `update:foo` 事件来同步。

```js
const app = Vue.createApp({})

app.component('my-component', {
  props: {
    foo: String
  },
  template: `
    <input
      type="text"
      :value="foo"
      @input="$emit('update:foo', $event.target.value)">
  `
})
```

```html
<my-component v-model:foo="bar"></my-component>
```

## 多个 `v-model` 绑定

通过利用特定 prop 和事件的能力，就像我们之前在上面 [`v-model` 参数中](#v-model-arguments)学习的那样，我们可以在一个组件实例上创建多个 `v-model` 实例。

每一个 `v-model` 都会与不同的 prop 相同步，而无需在组件内添加额外的属性。

```html
<user-name
  v-model:first-name="firstName"
  v-model:last-name="lastName"
></user-name>
```

```js
const app = Vue.createApp({})

app.component('user-name', {
  props: {
    firstName: String,
    lastName: String
  },
  template: `
    <input
      type="text"
      :value="firstName"
      @input="$emit('update:firstName', $event.target.value)">

    <input
      type="text"
      :value="lastName"
      @input="$emit('update:lastName', $event.target.value)">
  `
})
```

<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="result" data-user="shenqingchuan" data-slug-hash="XWdMmbP" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="多个 v-model">
  <span>See the Pen <a href="https://codepen.io/shenqingchuan/pen/XWdMmbP">
  多个 v-model</a> by shenqingchuan (<a href="https://codepen.io/shenqingchuan">@shenqingchuan</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

## 处理 `v-model` 修饰符

当在学习输入绑定时，我们看到了 `v-model` 有一些 [内建修饰符](/guide/forms.html#modifiers) - `.trim`, `.number` 和 `.lazy`。然而在某些场景下，你可能也想要自定义的修饰符。

让我们创建一个示例修饰符 `capitalize`，它会将 `v-model` 绑定提供的字符串首字母转为大写。

被添加到组件 `v-model` 上的修饰符会通过 `modelModifiers` prop 提供给组件，在下面的例子中，我们创建了一个包含 `modelModifiers` prop 的组件，该属性默认是一个空对象。

注意，当组件的 `created` 生命周期钩子触发时，该 `modelModifiers` prop 包含 `capitalize` 且它的值为 `true` - 由于我们在 `v-model` 绑定上设置了 `v-model.capitalize="bar"`.

```html
<my-component v-model.capitalize="bar"></my-component>
```

```js
app.component('my-component', {
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({})
    }
  },
  template: `
    <input type="text"
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)">
  `,
  created() {
    console.log(this.modelModifiers) // { capitalize: true }
  }
})
```

现在我们的 prop 设置完成，我们可以检查一下 `modelModifiers` 对象的各个属性，并为改变所抛出的值写一个处理器。下面的代码中，一旦 `<input />` 元素触发了一个 `input` 事件，就会使拿到的字符串首字母大写。

```html
<div id="app">
  <my-component v-model.capitalize="myText"></my-component>
  {{ myText }}
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      myText: ''
    }
  }
})

app.component('my-component', {
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({})
    }
  },
  methods: {
    emitValue(e) {
      let value = e.target.value
      if (this.modelModifiers.capitalize) {
        value = value.charAt(0).toUpperCase() + value.slice(1)
      }
      this.$emit('update:modelValue', value)
    }
  },
  template: `<input
    type="text"
    :value="modelValue"
    @input="emitValue">`
})

app.mount('#app')
```

对于带参数的 `v-model`，生成的 prop 名字将会是 `arg + "Modifiers"`：

```html
<my-component v-model:foo.capitalize="bar"></my-component>
```

```js
app.component('my-component', {
  props: ['foo', 'fooModifiers'],
  template: `
    <input type="text"
      :value="foo"
      @input="$emit('update:foo', $event.target.value)">
  `,
  created() {
    console.log(this.fooModifiers) // { capitalize: true }
  }
})
```
