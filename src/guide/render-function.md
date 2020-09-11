# 渲染函数

Vue 推荐在绝大多数情况下使用模板来创建你的 HTML。然而我们仍有一些场景需要 JavaScript 的灵活的编程式表现力。这时我们就可以使用**渲染函数**。

让我们看一个能体现 `render()` 函数实用性的例子，假设我们要生成一些带锚点的标题：

```html
<h1>
  <a name="hello-world" href="#hello-world">
    Hello world!
  </a>
</h1>
```

带锚点的标题十分常用，我们应该创建一个组件：

```vue-html
<anchored-heading :level="1">Hello world!</anchored-heading>
```

当开始写一个只能通过 `level` prop 动态生成标题 (heading) 的组件时，你可能很快想到这样实现：

```js
const app = Vue.createApp({})

app.component('anchored-heading', {
  template: `
    <h1 v-if="level === 1">
      <slot></slot>
    </h1>
    <h2 v-else-if="level === 2">
      <slot></slot>
    </h2>
    <h3 v-else-if="level === 3">
      <slot></slot>
    </h3>
    <h4 v-else-if="level === 4">
      <slot></slot>
    </h4>
    <h5 v-else-if="level === 5">
      <slot></slot>
    </h5>
    <h6 v-else-if="level === 6">
      <slot></slot>
    </h6>
  `,
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

这里用[模板](template-syntax.html)并不是最好的选择：不但代码冗长，而且在每一个级别的标题中重复书写了 `<slot></slot>`，在要插入锚点元素时还要再次重复。

虽然模板语法在大多数组件中都非常好用，但是显然在这里它就不合适了。那么，我们来尝试使用 `render()` 函数重写上面的例子：

```js
const app = Vue.createApp({})

app.component('anchored-heading', {
  render() {
    const { h } = Vue

    return h(
      'h' + this.level, // 标签名称
      {}, // props/attributes
      this.$slots.default() // 子组件数组
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

`render()` 方法看起来简洁多了！但是需要你非常熟悉组件实例的 property。在这个例子中，你需要知道，向组件中传递不带 `v-slot` 指令的子节点时，比如 `anchored-heading` 中的 `Hello world!`，这些子节点被存储在组件实例中的 `$slots.default` 中。如果你还不了解，**在深入渲染函数之前推荐阅读实例 [property API](../api/instance-properties.html)**。

## 节点、树以及虚拟 DOM

在深入渲染函数之前，重点先了解一些浏览器的工作原理。以下面这段 HTML 为例：

```html
<div>
  <h1>我的标题</h1>
  一些文本内容
  <!-- TODO: 添加些标语 -->
</div>
```

当浏览器读到这些代码时，它会建立一个 [“DOM 节点”树](https://javascript.info/dom-nodes)来帮助它追踪所有内容。

上述 HTML 对应的 DOM 节点树如下图所示：

![DOM Tree Visualization](/images/dom-tree.png)

每个元素都是一个节点。每段文字也是一个节点。甚至注释也都是节点。每个节点都可以有子节点 (即每个节点可以包含其它节点)。

高效地更新所有节点会是比较困难的，不过所幸你不必手动完成这个工作。你只需要告诉 Vue 你希望页面上的 HTML 是什么，在模板里：

```html
<h1>{{ blogTitle }}</h1>
```

或者在渲染函数里：

```js
render() {
  return Vue.h('h1', {}, this.blogTitle)
}
```

在这两种情况下，Vue 都会随着 `blogTitle` 发生改变而自动更新页面。

## 虚拟 DOM

Vue 通过让页面建立一个**虚拟 DOM**并追踪其自身的变化，来改变真实 DOM。请仔细阅读这行代码：

```js
return Vue.h('h1', {}, this.blogTitle)
```

`h()` 到底会返回什么呢？其实不是一个*实际的* DOM 元素。它返回一个普通对象，该对象包含向 Vue 描述其应在页面上呈现的节点类型的信息，包括所有子节点的描述。我们把这样的节点描述为“虚拟节点 (virtual node)”，也常简写它为“VNode” 。“虚拟 DOM”则是我们对由 Vue 组件树建立起来的整个 VNode 树的称呼。

## `h()` 参数

`h()` 函数的作用是创建 VNodes。也许称之为 `createVNode()` 更准确，但由于使用频繁，为了命名简洁将其定为 `h()`。它接受 3 个参数：

```js
// @returns {VNode}
h(
  // {String | Object | Function | null} tag
  // 一个 HTML 标签名、一个组件，一个动态组件，或者 null。
  // 如果是 null 则会生产一个注释
  //
  // 必填项
  'div',

  // {Object} props
  // 一个包含 attributes, props 和 events 的对象
  // 我们会在模板中使用
  //
  // 可选项
  {},

  // {String | Array | Object} children
  // 子级虚拟节点 (VNodes), 用 `h()` 构建而成，
  // 也可以使用字符串来生成“文本虚拟节点” 或者
  // 一个带slots的对象
  //
  // 可选项
  [
    'Some text comes first.',
    h('h1', 'A headline'),
    h(MyComponent, {
      someProp: 'foobar'
    })
  ]
)
```

## 完整示例

有了这些知识，我们现在可以完成我们最开始想实现的组件：

```js
const app = Vue.createApp({})

/** 递归地从子节点获取文本 */
function getChildrenTextContent(children) {
  return children
    .map(node => {
      return typeof node.children === 'string'
        ? node.children
        : Array.isArray(node.children)
        ? getChildrenTextContent(node.children)
        : ''
    })
    .join('')
}

app.component('anchored-heading', {
  render() {
    // 在子节点中创建 kebab-case 风格的 ID
    const headingId = getChildrenTextContent(this.$slots.default())
      .toLowerCase()
      .replace(/\W+/g, '-') // 用 - 代替非单词字符
      .replace(/(^-|-$)/g, '') // 移除前后 - 符号

    return Vue.h('h' + this.level, [
      Vue.h(
        'a',
        {
          name: headingId,
          href: '#' + headingId
        },
        this.$slots.default()
      )
    ])
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

## 约束

### VNode 必须唯一

组件树中的所有 VNode 必须是唯一的。这意味着，下面的渲染函数是不合法的：

```js
render() {
  const myParagraphVNode = Vue.h('p', 'hi')
  return Vue.h('div', [
    // 报错 - 重复的 VNodes!
    myParagraphVNode, myParagraphVNode
  ])
}
```

如果你真的需要重复很多次的元素/组件，你可以使用工厂函数来实现。例如，下面这渲染函数用完全合法的方式渲染了 20 个相同的段落：

```js
render() {
  return Vue.h('div',
    Array.apply(null, { length: 20 }).map(() => {
      return Vue.h('p', 'hi')
    })
  )
}
```

## 使用 JavaScript 代替模板功能

### `v-if` 和 `v-for`

只要在原生的 JavaScript 中可以轻松完成的操作，Vue 的渲染函数就不会提供专有的替代方法。比如，在模板中使用的 `v-if` 和 `v-for`：

```html
<ul v-if="items.length">
  <li v-for="item in items">{{ item.name }}</li>
</ul>
<p v-else>No items found.</p>
```

这些都可以在渲染函数中用 JavaScript 的 `if`/`else` 和 `map()` 来重写：

```js
props: ['items'],
render() {
  if (this.items.length) {
    return Vue.h('ul', this.items.map((item) => {
      return Vue.h('li', item.name)
    }))
  } else {
    return Vue.h('p', 'No items found.')
  }
}
```

### `v-model`

`v-model`指令可以在模版中用 props 属性 `modelValue`和`onUpdate:modelValue` 实现——我们必须自己实现相应的逻辑：

```js
props: ['modelValue'],
render() {
  return Vue.h(SomeComponent, {
    modelValue: this.modelValue,
    'onUpdate:modelValue': value => this.$emit('update:modelValue', value)
  })
}
```

### `v-on`

同时我们也需要实现事件所对应的 prop，比如，处理 `click` 事件，对应必须加入 prop 名称为 `onClick` 的事件处理。

```js
render() {
  return Vue.h('div', {
    onClick: $event => console.log('clicked', $event.target)
  })
}
```

#### 事件修饰符

对于 `.passive`，`.capture`，和 `.once` 的事件修饰符，Vue 提供了带有对象语法的处理函数：

比如:

```javascript
render() {
  return Vue.h('input', {
    onClick: {
      handler: this.doThisInCapturingMode,
      capture: true
    },
    onKeyUp: {
      handler: this.doThisOnce,
      once: true
    },
    onMouseOver: {
      handler: this.doThisOnceInCapturingMode,
      once: true,
      capture: true
    },
  })
}
```

对于所有其他事件和键修饰符，不需要特殊的 API，因为我们可以在处理程序中使用处理函数：

| 修饰器                                                 | 处理方法                                                                                                   |
| ----------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `.stop`                                               | `event.stopPropagation()`                                                                                  |
| `.prevent`                                            | `event.preventDefault()`                                                                                   |
| `.self`                                               | `if (event.target !== event.currentTarget) return`                                                         |
| 按键：<br>`.enter`, `.13`                              | `if (event.keyCode !== 13) return` (将 `13` 修改为[其他键盘 key](http://keycode.info/) 用于其他关键修饰符) |
| 修饰键：<br>`.ctrl`, `.alt`, `.shift`, `.meta` | `if (!event.ctrlKey) return` (将 `ctrlKey` 分别修改为 `altKey`，`shiftKey`，或者 `metaKey`)                |

下面是这些修饰词一起使用的示例：

```js
render() {
  return Vue.h('input', {
    onKeyUp: event => {
      // 如果触发事件的元素不是事件绑定的元素
      // 则返回
      if (event.target !== event.currentTarget) return
      // 如果按下去的不是 enter 键或者
      // 没有同时按下 shift 键
      // 则返回
      if (!event.shiftKey || event.keyCode !== 13) return
      // 阻止 事件冒泡
      event.stopPropagation()
      // 阻止该元素默认的 keyup 事件
      event.preventDefault()
      // ...
    }
  })
}
```

### 插槽

您可以从 [`this.$slots`](../api/instance-properties.html#slots) 将插槽内容作为 VNode 数组访问

```js
render() {
  // `<div><slot></slot></div>`
  return Vue.h('div', {}, this.$slots.default())
}
```

```js
props: ['message'],
render() {
  // `<div><slot :text="message"></slot></div>`
  return Vue.h('div', {}, this.$slots.default({
    text: this.message
  }))
}
```

使用渲染函数将插槽内容传递给组件：

```js
render() {
  // `<div><child v-slot="props"><span>{{ props.text }}</span></child></div>`
  return Vue.h('div', [
    Vue.h('child', {}, {
      // 使用对象形式传递 `slots`
      // 以 { name: props => VNode | Array <VNode> } 的形式
      default: (props) => Vue.h('span', props.text)
    })
  ])
}
```

## JSX

如果你写了很多 `render` 函数，可能会觉得下面这样的代码写起来很痛苦：

```js
Vue.h(
  'anchored-heading',
  {
    level: 1
  },
  [Vue.h('span', 'Hello'), ' world!']
)
```

特别是对应的模板如此简单的情况下：

```vue-html
<anchored-heading :level="1"> <span>Hello</span> world! </anchored-heading>
```

这就是为什么会有一个 [Babel 插件](https://github.com/vuejs/jsx-next)，用于在 Vue 中使用 JSX 语法，它可以让我们回到更接近于模板的语法上：

```jsx
import AnchoredHeading from './AnchoredHeading.vue'

new Vue({
  el: '#demo',
  render() {
    return (
      <AnchoredHeading level={1}>
        <span>Hello</span> world!
      </AnchoredHeading>
    )
  }
})
```

要了解更多关于 JSX 如何映射到 JavaScript，请阅读[使用文档](https://github.com/vuejs/jsx-next#installation)。

## 模板编译

Vue 的模板实际上被编译成了渲染函数。这是一个实现细节，通常不需要关心。但如果你想看看模板的功能具体是怎样被编译的，可能会发现会非常有意思。下面是一个使用 `Vue.compile` 来实时编译模板字符串的简单示例：

<iframe src="https://vue-next-template-explorer.netlify.app/" width="100%" height="420"></iframe>
