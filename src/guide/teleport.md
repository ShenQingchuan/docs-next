# Teleport

Vue 鼓励我们通过将 UI 和其相关行为封装到组件内的方式来构建用户界面。我们可以将它们彼此嵌套以构建生成一颗应用程序的 UI 树。

但是，有时候一部分组件模板虽然在逻辑上属于该组件，但从实现上来看，最好是将这部分模板移动到 Vue 应用外部的 DOM 上。

常见的情况是创建一个包含全屏模态框的组件。在大多数情况下，你希望模态框的逻辑写在组件中，但很快你会发现通过 CSS 或更改组件内的结构来解决模态框的定位问题是件很麻烦的事情。

考虑下面的 HTML 结构。

```html
<body>
  <div style="position: relative;">
    <h3>Tooltips with Vue 3 Teleport</h3>
    <div>
      <modal-button></modal-button>
    </div>
  </div>
</body>
```

让我们看看 `modal-button` 组件。

这个组件拥有一个可以打开模态框的 `button`，以及一个具有 `.modal` 类名的 `div` 元素，其中包含模态框的内容和关闭按钮。

```js
const app = Vue.createApp({});

app.component('modal-button', {
  template: `
    <button @click="modalOpen = true">
        Open full screen modal!
    </button>

    <div v-if="modalOpen" class="modal">
      <div>
        I'm a modal! 
        <button @click="modalOpen = false">
          Close
        </button>
      </div>
    </div>
  `,
  data() {
    return { 
      modalOpen: false
    }
  }
})
```

当在初始 HTML 结构中使用这个组件时，我们可以发现一个问题——模态框在深层嵌套的 `div` 中渲染且具有 `position: absolute` 定位的模态框相对于其父级 `div` 定位。

Teleport 提供了一个简单明了的方法，允许我们控制 DOM 节点在 HTML 哪部分结构中进行渲染，而不必借助于全局状态或将组件拆分成两部分。

让我们使用 `<teleport>` 修改 `modal-button` 组件并告诉 Vue “**传送**这部分 HTML 到 **body** 标签下”

```js
app.component('modal-button', {
  template: `
    <button @click="modalOpen = true">
        Open full screen modal! (With teleport!)
    </button>

    <teleport to="body">
      <div v-if="modalOpen" class="modal">
        <div>
          I'm a teleported modal! 
          (My parent is "body")
          <button @click="modalOpen = false">
            Close
          </button>
        </div>
      </div>
    </teleport>
  `,
  data() {
    return { 
      modalOpen: false
    }
  }
})
```

结果一旦我们点击按钮打开模态框时，Vue 会将模态框的内容作为 `body` 标签的子节点正确的渲染。

<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="result" data-user="shenqingchuan" data-slug-hash="ZEWvQpL" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Vue 3 Teleport">
  <span>See the Pen <a href="https://codepen.io/shenqingchuan/pen/ZEWvQpL">
  Vue 3 Teleport</a> by shenqingchuan (<a href="https://codepen.io/shenqingchuan">@shenqingchuan</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

## 与 Vue 组件一起使用

如果 `<teleport>` 包含一个 Vue 组件，该组件将成为 `<teleport>` 父级的逻辑子组件：

```js
const app = Vue.createApp({
  template: `
    <h1>Root instance</h1>
    <parent-component />
  `
})

app.component('parent-component', {
  template: `
    <h2>This is a parent component</h2>
    <teleport to="#endofbody">
      <child-component name="John" />
    </teleport>
  `
})

app.component('child-component', {
  props: ['name'],
  template: `
    <div>Hello, {{ name }}</div>
  `
})
```

在这个例子中，即使当 `child-component` 在不同的地方渲染，它作为 `parent-component` 的子组件仍然能接收到父组件传递的 `name` 属性。

这也意味着来自父组件的注入会按预期方式工作，并且在 Vue Devtools 中子组件将嵌套在父组件下方，而不是放置在实际被移动到的位置。

## 在同一个目标上使用多个 teleport

常见的使用场景是可复用且同时拥有多个活动实例的 `<Modal>` 组件。对于这种情况，多个 `<teleport>` 组件可以将其内容挂载到同一个目标元素下。将按照追加顺序拼接——在同一目标元素中，较晚挂载的元素将在较早挂载元素之后。

```html
<teleport to="#modals">
  <div>A</div>
</teleport>
<teleport to="#modals">
  <div>B</div>
</teleport>

<!-- result-->
<div id="modals">
  <div>A</div>
  <div>B</div>
</div>
```

你可以在 [API reference](../api/built-in-components.html#teleport) 中查看 `<teleport>` 组件的选项。
