# 计算属性与侦听器

> 本章节将使用 [单文件组件](single-file-component.html)语法作为代码示例。

## 计算值

有时我们需要一个状态依赖另一个状态——在 Vue 中是通过 [计算属性](computed.html#computed-properties) 进行处理。可直接创建一个计算属性值，我们直接使用 `computed` 方法：它是一个 getter 方法并返回一个只读的 [ref](reactivity-fundamentals.html#creating-standalone-reactive-values-as-refs) 对象。


```js
const count = ref(1)
const plusOne = computed(() => count.value + 1)

console.log(plusOne.value) // 2

plusOne.value++ // error
```

或者它也可以是含有 `get` 和 `set` 函数的对象，来创建一个可修改的 ref 对象。

```js
const count = ref(1)
const plusOne = computed({
  get: () => count.value + 1,
  set: val => {
    count.value = val - 1
  }
})

plusOne.value = 1
console.log(count.value) // 0
```

## `watchEffect`

要基于响应式状态更新或者自动更新，应使用 `watchEffect` 方法。它会通过立即执行来响应式地追踪记录依赖，并在依赖变化时重新运行该函数。

```js
const count = ref(0)

watchEffect(() => console.log(count.value))
// -> logs 0

setTimeout(() => {
  count.value++
  // -> logs 1
}, 100)
```

### 停止侦听器

当 `watchEffect` 声明在组件内的 [`setup()`](composition-api-setup.html) 函数或者 [生命周期钩子](composition-api-lifecycle-hooks.html)，侦听器会关联到生命周期，并且当组件销毁时会自动停止。

在一些情况下，也可以直接调用停止函数以停止侦听：

```js
const stop = watchEffect(() => {
  /* ... */
})

// later
stop()
```

### 清除副作用

有时副作用函数会执行一些异步的副作用, 这些响应需要在其失效时清除（即副作用完成之前状态已改变了）。所以侦听副作用传入的函数可以接收一个`onInvalidate` 函数之后，可用于注册清理无效的回调。当以下情况发生时，这个失效回调会被触发:

- 副作用即将重新执行时
- 侦听器被停止（例如：`watchEffect` 在 `setup()` 或生命周期钩子中被使用了，而此时组件即将卸载）

```js
watchEffect(onInvalidate => {
  const token = performAsyncOperation(id.value)
  onInvalidate(() => {
    // id 被改变或者侦听器停止。
    // 取消之前的异步操作
    token.cancel()
  })
})
```

因为返回值对于异步错误处理很重要，所以我们通过传入一个函数去注册失效回调，而不是从回调返回它。在执行数据请求时，副作用函数往往是一个异步函数：

```js
const data = ref(null)
watchEffect(async onInvalidate => {
  onInvalidate(() => {...}) // 在Promise resolves之前我们注册一个清除函数
  data.value = await fetchData(props.id)
})
```

异步函数隐式返回 Promise，但是在 Promise 完成之前，必须立即注册清除函数。此外，Vue 依靠该返回的 Promise 来自动处理 Promise 链中的潜在错误。

### 副作用刷新时机

Vue 的响应式系统会缓存副作用函数，并异步地刷新它们，这样可以避免同一个 “tick” 中多个状态改变导致的不必要的重复调用。在核心的具体实现中, 组件的 `update` 函数也是一个被侦听的副作用。当一个用户定义的副作用函数进入队列时, 会在所有的组件 `update` 副作用后执行：

```html
<template>
  <div>{{ count }}</div>
</template>

<script>
  export default {
    setup() {
      const count = ref(0)

      watchEffect(() => {
        console.log(count.value)
      })

      return {
        count
      }
    }
  }
</script>
```

在这个例子中：

- count 会在初始运行时同步打印出来
- 当 `count` 被更改时，将在组件**更新后**执行副作用。

注意： 第一次运行是在组件挂载完成之前，所以如果想在副作用中访问 DOM （或者模板 refs），则需要在 onMounted 钩子中。

```js
onMounted(() => {
  watchEffect(() => {
    // 通过 DOM 或者 模板 refs
  })
})
```

如果副作用需要同步或在组件更新之前重新运行，我们可以传递一个额外的 `options` 拥有 `flush` 属性的对象作为选项（默认为 `'post'`）：

```js
// 同步触发
watchEffect(
  () => {
    /* ... */
  },
  {
    flush: 'sync'
  }
)

// 组件更新前触发
watchEffect(
  () => {
    /* ... */
  },
  {
    flush: 'pre'
  }
)
```

### 侦听器调试

 `onTrack` 和 `onTrigger` 选项可用于调试侦听者的行为。
- 当依赖一个响应式的属性或者引用被追踪，`onTrack` 会被调用。
- 当观察者回调由依赖项的改变触发时，将调用 `onTrigger` 

这两个回调都将接收到一个包含有关所依赖项信息的调试器事件。建议在以下回调中编写 `debugger` 语句来检查依赖关系：

```js
watchEffect(
  () => {
    /* 副作用内容 */
  },
  {
    onTrigger(e) {
      debugger
    }
  }
)
```

`onTrack` 和 `onTrigger` 仅在开发环境下生效。

## `watch`

 `watch` API与组件 [watch](computed.html#watchers) property 完全等效。`watch` 需要侦听一个特殊的数据来源，并在单独的回调函数中应用副作用。即：仅在侦听的源更改时才调用此回调。

- 和 [`watchEffect`](#watcheffect) 相比较, `watch` 允许我们:

  - 懒执行副作用
  - 更明确哪些状态的改变会触发侦听器重新运行副作用
  - 访问侦听状态变化前后的值


### 侦听单个数据源

侦听器的数据源可以是一个拥有返回值的 getter 函数，也可以是 `ref`：

```js
// 侦听一个 getter
const state = reactive({ count: 0 })
watch(
  () => state.count,
  (count, prevCount) => {
    /* ... */
  }
)

// 直接侦听一个 ref
const count = ref(0)
watch(count, (count, prevCount) => {
  /* ... */
})
```

### 侦听多个数据源

侦听器也可以使用数组来同时侦听多个源：

```js
watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {
  /* ... */
})
```

### 和 `watchEffect` 共享行为

`watch` 和 [`watchEffect`](#watcheffect) 在 [停止侦听](#stopping-the-watcher), [清除副作用](#side-effect-invalidation) (相应地 `onInvalidate` 会作为回调的第三个参数传入)，[副作用刷新时机](#effect-flush-timing) 和 [侦听器调试](#watcher-debugging) 等方面行为一致。
