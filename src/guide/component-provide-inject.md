# Provide / inject

> 该页面假设你已经阅读过了[组件基础](component-basics.md)。如果你还对组件不太了解，推荐你先阅读它。

通常，当我们需要从父组件传递数据到子组件时，我们会使用 [props](component-props.md)。想象我们有如图所示的一个层次结构很深的组件而你只需要将一段数据从父组件传递到最深层的叶子组件。在这个案例中，如果你仍然采用 prop 从上向下传递将会很困扰。

对于这样的需求，我们应该使用 `provide` 和 `inject` 这一对。父组件能够作为它所有子组件的依赖提供者，无论向下层级有多深。这个功能分为两个部分，父组件有一个 `provide` 选项来提供数据，而子组件有一个 `inject` 选项来注入、使用该数据。

![Provide/inject 结构](/images/components_provide.png)

例如，我们有一个这样的应用结构：

```
Root
└─ TodoList
   ├─ TodoItem
   └─ TodoListFooter
      ├─ ClearTodosButton
      └─ TodoListStatistics
```

如果我们想要传递待办事项列表的长度给 `TodoListStatistics`，我们将要将该 prop 传递过整个层次结构：`TodoList` -> `TodoListFooter` -> `TodoListStatistics`。若采用 provide/inject 我们可以直接这样做：

```js
const app = Vue.createApp({})

app.component('todo-list', {
  data() {
    return {
      todos: ['喂猫', '买票']
    }
  },
  provide: {
    user: 'John Doe'
  },
  template: `
    <div>
      {{ todos.length }}
      <!-- rest of the template -->
    </div>
  `
})

app.component('todo-list-statistics', {
  inject: ['user'],
  created() {
    console.log(`注入 property: ${this.user}`) // > 注入 property: John Doe
  }
})
```

然而，当我们尝试提供一些组件实例的 property 时却没有效果：

```js
app.component('todo-list', {
  data() {
    return {
      todos: ['Feed a cat', 'Buy tickets']
    }
  },
  provide: {
    todoLength: this.todos.length // 这将会抛出错误 'Cannot read property 'length' of undefined`
  },
  template: `
    ...
  `
})
```

要想访问到组件实例的 property，我们需要将 `provide` 转为一个返回对象的函数：

```js
app.component('todo-list', {
  data() {
    return {
      todos: ['喂猫', '买票']
    }
  },
  provide() {
    return {
      todoLength: this.todos.length
    }
  },
  template: `
    ...
  `
})
```

这允许我们更安全地继续开发该组件，而不用担心我们可能会更改/删除子组件所依赖的东西。这些组件之间的接口仍然被清晰地定义，就像 props 一样。

事实上，你可以把依赖注入想成是一种 “长距离 props”：

- 父组件无需知道哪一个子组件要使用它提供的 property
- 子组件同样无需知道注入的 property 从何而来

## 与响应式协同工作

在上面的例子中，如果我们更改了 `todos` 列表，该更改将不会反射到已经注入的 `todoLength` property 上。这是由于 `provide/inject` 绑定默认 _不是_ 响应式的。我们可以通过传递一个 `ref` property 或者 `reactive` 对象给 `provide` 来改变此行为。
在此例中，如果我们想要对祖先组件的变化协同响应，我们应该通过组合式 API `computed` 包装我们提供的 `todolength` 。

```js
app.component('todo-list', {
  // ...
  provide() {
    return {
      todoLength: Vue.computed(() => this.todos.length)
    }
  }
})
```

这样任何对 `todos.length` 的变更都将会正确反馈到各个 `todoLength` 被注入的组件中。你可以在 [组合式 API 章节](composition-api-provide-inject.html#injection-reactivity)中读到更多关于 `reactive` provide/inject 的内容。
