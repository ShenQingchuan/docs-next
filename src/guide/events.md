# 事件处理

<common-dcloud-video href="https://learning.dcloud.io/#/?vid=10" />

## 监听事件

我们可以使用 `v-on` 指令，或是简写为 `@` 符号，来监听 DOM 事件，并在触发时运行一些 JavaScript 代码。用法如 `v-on:click="methodName"` 或是采用缩写， `@click="methodName"`

例如：

```html
<div id="basic-event">
  <button @click="counter += 1">+ 1</button>
  <p>上面这个按钮已经被点击了 {{ counter }} 次。</p>
</div>
```

```js
Vue.createApp({
  data() {
    return {
      counter: 1
    }
  }
}).mount('#basic-event')
```

结果：

<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="css,result" data-user="shenqingchuan" data-slug-hash="YzqpMpe" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="事件处理：基础">
  <span>See the Pen <a href="https://codepen.io/shenqingchuan/pen/YzqpMpe">
  事件处理：基础</a> by shenqingchuan (<a href="https://codepen.io/shenqingchuan">@shenqingchuan</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

## 事件处理方法

然而许多事件处理逻辑会更为复杂，所以直接把 JavaScript 代码写在 `v-on` 指令中是不可行的。因此 `v-on` 还可以接收一个需要调用的方法名称。

示例：

```html
<div id="event-with-method">
  <!-- `greet` 是在下面定义的方法名 -->
  <button @click="greet">打招呼</button>
</div>
```

```js
Vue.createApp({
  data() {
    return {
      name: 'Vue.js'
    }
  },
  methods: {
    // 在 `methods` 对象中定义方法
    greet(event) {
      // `this` 在方法里指向当前 Vue 实例
      alert('Hello ' + this.name + '!')
      // `event` 是原生 DOM 事件
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
}).mount('#event-with-method')
```

结果：

<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="js,result" data-user="shenqingchuan" data-slug-hash="eYZBogJ" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="事件处理：使用方法">
  <span>See the Pen <a href="https://codepen.io/shenqingchuan/pen/eYZBogJ">
  事件处理：使用方法</a> by shenqingchuan (<a href="https://codepen.io/shenqingchuan">@shenqingchuan</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

## 内联处理器中的方法

除了直接绑定到一个方法，也可以在内联 JavaScript 语句中调用方法：

```html
<div id="inline-handler">
  <button @click="say('你好')">说：你好</button>
  <button @click="say('什么鬼')">说：什么鬼</button>
</div>
```

```js
Vue.createApp({
  methods: {
    say(message) {
      alert(message)
    }
  }
}).mount('#inline-handler')
```

结果：

<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="css,result" data-user="shenqingchuan" data-slug-hash="poyNBeo" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="事件处理：内联处理方法">
  <span>See the Pen <a href="https://codepen.io/shenqingchuan/pen/poyNBeo">
  事件处理：内联处理方法</a> by shenqingchuan (<a href="https://codepen.io/shenqingchuan">@shenqingchuan</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

有时也需要在内联语句处理器中访问原始的 DOM 事件。可以用特殊变量 `$event` 把它传入方法：

```html
<button @click="warn('表单现在无法提交！', $event)">
  提交
</button>
```

```js
// ...
methods: {
  warn(message, event) {
    // 现在我们可以访问原生事件对象
    if (event) {
      event.preventDefault()
    }
    alert(message)
  }
}
```

## 多事件处理器

你可以像这样定义多个方法作为事件处理器，采用逗号分隔：

```html
<!-- one() 和 two() 在按钮被点击时都会执行 -->
<button @click="one($event), two($event)">
  提交
</button>
```

```js
// ...
methods: {
  one(event) {
    // 第一个处理方法多逻辑...
  },
  two(event) {
    // 第二个处理方法多逻辑...
  }
}
```

## 事件修饰符

在事件处理器中调用 `event.preventDefault()` or `event.stopPropagation()` 是一个很常见的需求，尽管我们可以很容易地在方法中实现，但方法仅与数据逻辑相关显然比还要处理 DOM 事件细节更好。

为了解决此类问题，Vue 为 `v-on` 提供了 **事件修饰符**，作为其指令的后缀。

- `.stop`
- `.prevent`
- `.capture`
- `.self`
- `.once`
- `.passive`

```html
<!-- 点击事件的 propagation 会被停止 -->
<a @click.stop="doThis"></a>

<!-- submit 事件将不会重新载入该页面 -->
<form @submit.prevent="onSubmit"></form>

<!-- 修饰符可以链式调用 -->
<a @click.stop.prevent="doThat"></a>

<!-- 只使用修饰符 -->
<form @submit.prevent></form>

<!-- 当添加监听器时使用 capture 模式 -->
<!-- 例如：一个内层元素事件的事件会先被外层处理 -->
<div @click.capture="doThis">...</div>

<!-- 只在 event.target 是此元素本身时触发事件处理器 -->
<!-- 例如：不接受子元素的事件 -->
<div @click.self="doThat">...</div>
```

::: tip 注意
使用修饰符时，顺序很重要，因为相关代码是按照相同的顺序生成的。因此使用 `@click.prevent.self` 将阻止所有点击，而 `@click.self.prevent` 只会阻止对元素本身的点击。
:::

```html
<!-- 该点击事件最多被触发一次 -->
<a @click.once="doThis"></a>
```

与其他专用于原生 DOM 事件的修饰符不同，`.once` 修饰符也可以用于组件事件。如果您还没有读过[组件章节](component-custom-events.html)，那么现在暂时不用担心这个问题。

Vue 同时提供了 `.passive` 修饰符，对应于 [`addEventListener` 的 `passive` 选项](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#Parameters).

```html
<!-- 滚动事件的默认行为 (指滚动) 会立即发生 -->
<!-- 而不是等到 `onScroll` 完成  -->
<!-- 因为其包含了 `event.preventDefault()` -->
<div @scroll.passive="onScroll">...</div>
```

`.passive` 修饰符对提升移动端设备上对体验十分有用。

::: tip 注意
不要同时使用 `.passive` 和 `.prevent`，因为 `.prevent` 将被忽略并且浏览器可能会显示一个警告， 请记住，`.passive` 会告诉浏览器你不想阻止事件的默认行为。
:::

## 键盘修饰符

当监听键盘事件时，我们经常需要去检查一些具体的键。Vue 允许在监听键盘输入事件时给 `v-on` 或 `@` 添加键盘修饰符:

```html
<!-- `vm.submit()` 只会在键盘输入的键 `key` 是 `Enter` 时执行 -->
<input @keyup.enter="submit" />
```

你可以直接使用 [`KeyboardEvent.key`] 暴露的任何有效键名，转为短横线连接的 kabab-case 格式来形成一个修饰符。

```html
<input @keyup.page-down="onPageDown" />
```

在上面的例子中，事件处理起仅会在 `$event.key` 为 `'PageDown'` 时被调用。

### 键位别名

Vue 为大部分常用键位都提供了别名绑定：

- `.enter`
- `.tab`
- `.delete` (同时包括 "Delete" 和 "Backspace" 两个键)
- `.esc`
- `.space`
- `.up`
- `.down`
- `.left`
- `.right`

## 系统修饰键

可以用如下修饰符来实现仅在按下相应按键时才触发鼠标或键盘事件的监听器。

- `.ctrl`
- `.alt`
- `.shift`
- `.meta`

::: tip 注意
注意：在 Mac 系统键盘上，meta 对应 command 键 (⌘)。在 Windows 系统键盘 meta 对应 Windows 徽标键 (⊞)。在 Sun 操作系统键盘上，meta 对应实心宝石键 (◆)。在其他特定键盘上，尤其在 MIT 和 Lisp 机器的键盘、以及其后继产品，比如 Knight 键盘、space-cadet 键盘，meta 被标记为“META”。在 Symbolics 键盘上，meta 被标记为“META”或者“Meta”。
:::

例如：

```html
<!-- Alt + Enter -->
<input @keyup.alt.enter="clear" />

<!-- Ctrl + Click -->
<div @click.ctrl="doSomething">...</div>
```

::: tip 注意
请注意修饰键与常规按键不同，在和 `keyup` 事件一起用时，事件触发时修饰键必须处于按下状态。换句话说，只有在按住 `ctrl` 的情况下释放其它按键，才能触发 `keyup.ctrl`。而单单释放 `ctrl` 也不会触发事件。如果你想要这样的行为，请为 `ctrl` 换用 `keyCode`：`keyup.17`。
:::

### `.exact` 修饰符

`.exact` 修饰符允许你控制由精确的系统修饰符组合触发的事件。

```html
<!-- 即使 Alt 或 Shift 被一同按下时也会触发 -->
<button @click.ctrl="onClick">A</button>

<!-- 有且只有 Ctrl 被按下的时候才触发 -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- 没有任何系统修饰符被按下的时候才触发 -->
<button @click.exact="onClick">A</button>
```

### 鼠标按钮修饰符

- `.left`
- `.right`
- `.middle`

这些修饰符会限制处理函数仅响应特定的鼠标按钮。

## 为什么在 HTML 中监听事件

你可能注意到这种事件监听的方式违背了关注点分离 (separation of concern) 这个长期以来的优良传统。但不必担心，因为所有的 Vue.js 事件处理方法和表达式都严格绑定在当前视图的 ViewModel 上，它不会导致任何维护上的困难。实际上，使用 v-on 有几个好处：

1. 扫一眼 HTML 模板便能轻松定位在 JavaScript 代码里对应的方法。

2. 因为你无须在 JavaScript 里手动绑定事件，你的 ViewModel 代码可以是非常纯粹的逻辑，和 DOM 完全解耦，更易于测试。

3. 当一个 ViewModel 被销毁时，所有的事件处理器都会自动被删除。你无须担心如何清理它们。
