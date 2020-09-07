# 非 Prop 的 Attribute

> 该页面假设你已经阅读过了[组件基础](component-basics.md)。如果你还对组件不太了解，推荐你先阅读它。

一个非 `prop` 的 attribute 是指传向一个组件，但是该组件并没有在 [prop](component-props) 或 [emits](component-custom-events.html#defining-custom-events) 中定义的 attribute。常见的例子如：`class`，`style` 和 `id` 这些 attribute

## Attribute 继承

当组件返回单个根节点时，非 prop 的属性将自动添加到根节点的 attribute 中。例如，在日期选择器组件的实例中:

```js
app.component('date-picker', {
  template: `
    <div class="date-picker">
      <input type="datetime" />
    </div>
  `
})
```

在事件中，我们需要通过一个 `data-status` property 来定义日期选择器组件的状态，它将应用于根节点(即 `div.date-picker`)。

```html
<!-- 带有非 prop 属性的日期选择器组件 -->
<date-picker data-status="activated"></date-picker>

<!-- 渲染完成的日期选择器组件 -->
<div class="date-picker" data-status="activated">
  <input type="datetime" />
</div>
```

同样的规则适用于事件监听器:

```html
<date-picker @change="submitChange"></date-picker>
```

```js
app.component('date-picker', {
  created() {
    console.log(this.$attrs) // { onChange: () => {}  }
  }
})
```

当我们有一个带有 `change` 事件的 HTML 元素作为 `date-picker` 的根元素时，这可能会很有帮助。

```js
app.component('date-picker', {
  template: `
    <select>
      <option value="1">昨天</option>
      <option value="2">今天</option>
      <option value="3">明天</option>
    </select>
  `
})
```

在这种情况下，`change` 事件侦听器从父组件传递到子组件，它将在原生 `select` 元素的`change` 事件上被触发。我们不需要从 `date-picker` 显式触发一个事件:

```html
<div id="date-picker" class="demo">
  <date-picker @change="showChange"></date-picker>
</div>
```

```js
const app = Vue.createApp({
  methods: {
    showChange(event) {
      console.log(event.target.value) // will log a value of the selected option
    }
  }
})
```

## 禁用 Attribute 继承

如果你不希望组件的根元素继承 attribute，你可以在组件的选项中设置 `inheritAttrs: false`。例如：

禁用属性继承的常见场景是需要将属性应用于根节点之外的其他元素。

通过设置 `inheritAttrs` 选项为 `false`，使你能够访问到 `$attrs` property，它包含了所有没有写在组件 `props` 和 `emits` 里的 attribute（例如：`class`, `style`, `v-on` 监听器等等）。

当使用我们[上一章节](#attribute-inheritance)的日期选择器组件示例时，对于需要将所有非 prop 属性应用到 `input` 元素而不是根元素 `div` 的情况，可以通过使用 `v-bind` 快捷方式来完成。

```js{5}
app.component('date-picker', {
  inheritAttrs: false,
  template: `
    <div class="date-picker">
      <input type="datetime" v-bind="$attrs" />
    </div>
  `
})
```

使用这样的配置方式，我们的 `data-status` attribute 将会被应用在 `input` 元素上！

```html
<!-- 带有非 prop attribute 的日期选择器组件 -->
<date-picker data-status="activated"></date-picker>

<!-- 渲染完成的日期选择器组件 -->
<div class="date-picker">
  <input type="datetime" data-status="activated" />
</div>
```

## 多根节点上的 Attribute 继承

与单个根节点组件不同，具有多个根节点的组件没有自动直落行为。如果没有显式绑定 `$attrs`，将会发出一个运行时警告。

```html
<custom-layout id="custom-layout" @click="changeValue"></custom-layout>
```

```js
// 这将会抛出一个警告
app.component('custom-layout', {
  template: `
    <header>...</header>
    <main>...</main>
    <footer>...</footer>
  `
})

// 没有警告，$attrs 被传递给了 main 元素
app.component('custom-layout', {
  template: `
    <header>...</header>
    <main v-bind="$attrs">...</main>
    <footer>...</footer>
  `
})
```
