# Setup

> 本章节将使用 [单文件组件](single-file-component.html)语法作为代码示例。
> 这篇假设你已经阅读过了 [组合式 API](composition-api-introduction.html) 和 [响应式原理](reactivity-fundamentals.html)，如果你还对组合式 API 不太了解，推荐你先阅读它。

## 参数

使用 `setup` 方法需要两个参数：

1. `prop`
2. `context`

下面让我们来深入了解一下这两个参数的用法。

### prop

`setup` 函数中的第一个参数是 `prop` 。如同普通组件中的一样， `setup` 中的 `prop` 是响应式的，并且在传入新的 prop 时会被更新。

```js
// MyBook.vue

export default {
  prop: {
    title: String
  },
  setup(prop) {
    console.log(prop.title)
  }
}
```

::: warning 警告
因为 `prop` 具有响应性，所以你**不可以使用 ES6 的解构赋值**，这将会丢失其响应性！
:::

你可以使用 `setup` 函数中的 [toRefs](reactivity-fundamentals.html#destructuring-reactive-state)安全地对 prop 使用解构赋值。

```js
// MyBook.vue

import { toRefs } from 'vue'

setup(prop) {
 const { title } = toRefs(prop)

 console.log(title.value)
}
```

### Context

传递给 `setup` 函数的第二个参数是 `context` 。 `context` 是一个普通的 JavaScript 对象，它有三个组件 property：

```js
// MyBook.vue

export default {
  setup(prop, context) {
    // Attributes（非响应式对象）
    console.log(context.attrs)

    // 插槽（非响应式对象）
    console.log(context.slots)

    // 抛出事件（方法）
    console.log(context.emit)
  }
}
```

`context` 是一个普通的 JavaScript 对象，即说明这个对象不具有响应性，你可以放心大胆地对 `context` 使用解构赋值。

```js
// MyBook.vue
export default {
  setup(prop, { attrs, slots, emit }) {
    ...
  }
}
```

`attrs` 和 `slots` 是有状态的对象，在组件本身更新时同步更新。你应该避免它们对他们结构赋值，并按 `attrs.x` 或 `slots.x` 的方式使用它们。与 `props` 不同的是， `attrs` 和 `slots` 这两个 property 不具有**响应性**。如果你想基于 `attrs` 或 `slots` 的变化应用一些副作用，则应在 `onUpdated` 生命周期钩子中进行。

## 访问组件 property

执行 `setup` 时，组件实例尚未被创建。 此时你只能访问以下 property：

- `prop`
- `attrs`
- `slots`
- `emit`

也就是说你没法访问以下组件选项：

- `data`
- `computed`
- `methods`

## 配合模版使用

若 `setup` 返回的是一个对象，则可以在组件模板中访问该对象的 property：

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

请注意，从 `setup` 中返回 [refs](../api/refs-api.html#ref) 在模板中访问时会被 [自动解构](../api/refs-api.html#access-in-templates)，因此你不应在模板中使用 `.value`。

## 配合渲染函数使用

`setup` 还可以返回一个 render 函数，该函数可以直接使用在相同作用域内声明的响应式状态：

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

**在 `setup()` 内部，`this` 将不会引用当前活动实例**，由于 `setup()` 在其他组件选项解析之前被调用，因此 `setup()` 内部的 `this` 将与在其他选项中 `this` 的选项完全不同。 与其他选项式 API 一起使用 `setup()` 可能会引起混淆。
