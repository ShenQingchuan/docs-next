# Setup函数

> 本章节将使用[单文件组件](single-file-component.html)语法作为代码示例。

> 这篇假设你已经阅读过了[组合式API](composition-api-introduction.html)和[响应式原理](reactivity-fundamentals.html)，如果你还对组合式API不太了解，推荐你先阅读它。

## 参数

使用 `setup` 方法需要两个参数：

1. `props`
2. `context`

下面让我们来深入了解一下这两个参数的用法。

### Props

`setup` 函数中的第一个参数是 `props` 。如同普通组件中的一样， `setup` 中的 `props` 是响应式的，并且在传入新的props时会被更新。

```js
// MyBook.vue

export default {
  props: {
    title: String
  },
  setup(props) {
    console.log(props.title)
  }
}
```

::: warning 警告
因为 `props` 具有响应性，所以你**不可以使用ES6的解构赋值**，这将会丢失其相应性！
:::

你可以使用 `setup` 函数中的[toRefs](reactivity-fundamentals.html#destructuring-reactive-state)安全地对props使用解构赋值。

```js
// MyBook.vue

import { toRefs } from 'vue'

setup(props) {
	const { title } = toRefs(props)

	console.log(title.value)
}
```

### Context

传递给 `setup` 函数的第二个参数是 `context` 。 `context` 是一个普通的JavaScript对象，它有三个组件属性：

```js
// MyBook.vue

export default {
  setup(props, context) {
    // 属性 （非响应式对象）
    console.log(context.attrs)

    // 插槽 （非响应式对象）
    console.log(context.slots)

    // 触发事件 (方法)
    console.log(context.emit)
  }
}
```

`context` 是一个普通的JavaScript对象，即说明这个对象不具有响应性，你可以放心大胆地对 `context` 使用解构赋值。

```js
// MyBook.vue
export default {
  setup(props, { attrs, slots, emit }) {
    ...
  }
}
```

`attrs` 和 `slots` 是有状态的对象，在组件本身更新时总是会更新。 你应该避免它们对他们结构赋值，并按 `attrs.x` 或 `slots.x` 的方式使用它们。与 `props` 不同的是， `attrs` 和 `slots` 这两个属性不具有**响应性**。如果你想基于 `attrs` 或 `slots` 的更改使用副作用(side effects)，则应在 `onUpdated` 生命周期钩子中进行。

## 访问组件属性

执行 `setup` 时，组件实例尚未被创建。 此时你只能访问以下属性：

- `props`
- `attrs`
- `slots`
- `emit`

也就是说你没法访问以下组件选项：

- `data`
- `computed`
- `methods`

## 配合模版使用

若 `setup` 返回的是一个对象，则可以在组件模板中访问该对象的属性：

```vue-html
<!-- MyBook.vue -->
<template>
  <div>{{ readersNumber }} {{ book.title }}</div>
</template>

<script>
  import { ref, reactive } from 'vue'

  export default {
    setup() {
      const readersNumber = ref(0)
      const book = reactive({ title: 'Vue 3 Guide' })

      // expose to template
      return {
        readersNumber,
        book
      }
    }
  }
</script>
```

请注意，从 `setup` 中返回[refs](../api/refs-api.html#ref)在模板中访问时会被[自动解构](../api/refs-api.html#access-in-templates)，因此你不应在模板中使用 `.value`。

## 配合渲染函数使用

`setup` 还可以返回一个render函数，该函数可以直接使用在相同作用域内声明的响应式状态：

```js
// MyBook.vue

import { h, ref, reactive } from 'vue'

export default {
  setup() {
    const readersNumber = ref(0)
    const book = reactive({ title: 'Vue 3 Guide' })
    // 请注意，我们需要在此处暴露ref的值
    return () => h('div', [readersNumber.value, book.title])
  }
}
```

## 配合 `this` 使用

**在 `setup()` 内部，`this` 将不会引用当前活动实例**由于 `setup()` 在其他组件选项解析之前被调用，因此 `setup()` 内部的 `this` 将与在其他选项中 `this` 的选项完全不同。 与其他选项API一起使用 `setup()` 可能会引起混淆。