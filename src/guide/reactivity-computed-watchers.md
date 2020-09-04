~# 计算属性与侦听器

> This section uses [single-file component](single-file-component.html) syntax for code examples

> 这部分的示例代码采用的是[单页面组件](single-file-component.html)语法
## Computed values

Sometimes we need state that depends on other state - in Vue this is handled with component [computed properties](computed.html#computed-properties). To directly create a computed value, we can use the `computed` method: it takes a getter function and returns an immutable reactive [ref](reactivity-fundamentals.html#creating-standalone-reactive-values-as-refs) object for the returned value from the getter.

有时我们需要一个状态依赖另一个状态 -在vue中是通过[computed properties](computed.html#computed-properties)组件进行处理。可直接创建一个计算属性值，我们直接使用computed 方法：它是一个getter方法并返回一个只读的[ref](reactivity-fundamentals.html#creating-standalone-reactive-values-as-refs)对象。


```js
const count = ref(1)
const plusOne = computed(() => count.value + 1)

console.log(plusOne.value) // 2

plusOne.value++ // error
```

Alternatively, it can take an object with `get` and `set` functions to create a writable ref object.

或者它也可以是含有`get`和`set`函数的object，来创建一个可修改的ref object。

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

To apply and _automatically re-apply_ a side effect based on reactive state, we can use the `watchEffect` method. It runs a function immediately while reactively tracking its dependencies and re-runs it whenever the dependencies are changed.

基于响应状态更新或者自动更新，我们能使用`watchEffect`方法。它会响应式的追踪依赖，并在依赖变化时运行一个立即执行函数。

```js
const count = ref(0)

watchEffect(() => console.log(count.value))
// -> logs 0

setTimeout(() => {
  count.value++
  // -> logs 1
}, 100)
```

### Stopping the Watcher

When `watchEffect` is called during a component's [setup()](composition-api-setup.html) function or [lifecycle hooks](composition-api-lifecycle-hooks.html), the watcher is linked to the component's lifecycle and will be automatically stopped when the component is unmounted.

In other cases, it returns a stop handle which can be called to explicitly stop the watcher:


当 `watchEffect` 声明在组件内的 [setup()](composition-api-setup.html)函数或者[生命周期钩子](composition-api-lifecycle-hooks.html)，监听器会关联到生命周期，并且当组件销毁时会自动停止。

在一些情况下，也可以直接调用停止函数以停止侦听：

```js
const stop = watchEffect(() => {
  /* ... */
})

// later
stop()
```

### Side Effect Invalidation

Sometimes the watched effect function will perform asynchronous side effects that need to be cleaned up when it is invalidated (i.e state changed before the effects can be completed). The effect function receives an `onInvalidate` function that can be used to register an invalidation callback. This invalidation callback is called when:

- the effect is about to re-run
- the watcher is stopped (i.e. when the component is unmounted if `watchEffect` is used inside `setup()` or lifecycle hooks) 

有时副作用函数会执行一些异步的副作用, 这些响应需要在其失效时清除（即完成之前状态已改变了）。所以侦听副作用传入的函数可以接收一个`onInvalidate` 函数之后，可用于注册清理无效的回调。当以下情况发生时，这个失效回调会被触发:

- 副作用即将重新执行时
- 监听器被停止（如果在 `setup()` 或 生命周期钩子函数中使用了 `watchEffect` , 则在卸载组件时触发）

```js
watchEffect(onInvalidate => {
  const token = performAsyncOperation(id.value)
  onInvalidate(() => {
    // id has changed or watcher is stopped.
    // invalidate previously pending async operation
    token.cancel()
  })
})
```

We are registering the invalidation callback via a passed-in function instead of returning it from the callback because the return value is important for async error handling. It is very common for the effect function to be an async function when performing data fetching:

因为返回值对于异步错误处理很重要，所以我们通过传入一个函数去注册失效回调，而不是从回调返回它。在执行数据请求时，副作用函数往往是一个异步函数：

```js
const data = ref(null)
watchEffect(async onInvalidate => {
  onInvalidate(() => {...}) // we register cleanup function before Promise resolves
  data.value = await fetchData(props.id)
})
```

An async function implicitly returns a Promise, but the cleanup function needs to be registered immediately before the Promise resolves. In addition, Vue relies on the returned Promise to automatically handle potential errors in the Promise chain.

异步函数隐式返回Promise，但是在Promise解析之前，必须立即注册清除函数。此外，Vue依靠返回的Promise来自动处理Promise链中的潜在错误。

### Effect Flush Timing

Vue's reactivity system buffers invalidated effects and flushes them asynchronously to avoid unnecessary duplicate invocation when there are many state mutations happening in the same "tick". Internally, a component's `update` function is also a watched effect. When a user effect is queued, it is always invoked after all component `update` effects:

Vue 的响应式系统会缓存副作用函数，并异步地刷新它们，这样可以避免同一个 tick 中多个状态改变导致的不必要的重复调用。在核心的具体实现中, 组件的更新函数也是一个被侦听的副作用。当一个用户定义的副作用函数进入队列时, 会在所有的组件更新后执行：

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

In this example:

- The count will be logged synchronously on initial run.
- When `count` is mutated, the callback will be called **after** the component has updated.

Note the first run is executed before the component is mounted. So if you wish to access the DOM (or template refs) in a watched effect, do it in the mounted hook:

在这个例子中：

- count 会在初始运行时同步打印出来
- 当 count 被更改时，将在组件更新后执行副作用。

注意： 第一次运行是在组件 mounted 之前，所以如果想通过 DOM （或者 模板 refs）观察副作用请在 onMounted 钩子里。

```js
onMounted(() => {
  watchEffect(() => {
    // access the DOM or template refs
  })
})
```

In cases where a watcher effect needs to be re-run synchronously or before component updates, we can pass an additional `options` object with the `flush` option (default is `'post'`):

如果副作用需要同步或在组件更新之前重新运行，我们可以传递一个额外的 `options`拥有 `flush` 属性的对象作为选项（默认为 `'post'`）：

```js
// fire synchronously
watchEffect(
  () => {
    /* ... */
  },
  {
    flush: 'sync'
  }
)

// fire before component updates
watchEffect(
  () => {
    /* ... */
  },
  {
    flush: 'pre'
  }
)
```

### Watcher Debugging

The `onTrack` and `onTrigger` options can be used to debug a watcher's behavior.

- `onTrack` will be called when a reactive property or ref is tracked as a dependency
- `onTrigger` will be called when the watcher callback is triggered by the mutation of a dependency

Both callbacks will receive a debugger event which contains information on the dependency in question. It is recommended to place a `debugger` statement in these callbacks to interactively inspect the dependency:


 `onTrack` 和 `onTrigger`选项可用于调试观察者的行为。
- 当依赖一个响应式的属性或者引用被追踪，`onTrack` 会被调用。
-当观察者回调由依赖项的改变触发时，将调用 `onTrack` 

```js
watchEffect(
  () => {
    /* side effect */
  },
  {
    onTrigger(e) {
      debugger
    }
  }
)
```

`onTrack` and `onTrigger` only work in development mode.

## `watch`

The `watch` API is the exact equivalent of the component [watch](computed.html#watchers) property. `watch` requires watching a specific data source and applies side effects in a separate callback function. It also is lazy by default - i.e. the callback is only called when the watched source has changed.

- Compared to [watchEffect](#watcheffect), `watch` allows us to:

  - Perform the side effect lazily;
  - Be more specific about what state should trigger the watcher to re-run;
  - Access both the previous and current value of the watched state.

### Watching a Single Source

A watcher data source can either be a getter function that returns a value, or directly a `ref`:

```js
// watching a getter
const state = reactive({ count: 0 })
watch(
  () => state.count,
  (count, prevCount) => {
    /* ... */
  }
)

// directly watching a ref
const count = ref(0)
watch(count, (count, prevCount) => {
  /* ... */
})
```

### Watching Multiple Sources

A watcher can also watch multiple sources at the same time using an array:

```js
watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {
  /* ... */
})
```

### Shared Behavior with `watchEffect`

`watch` shares behavior with [`watchEffect`](#watcheffect) in terms of [manual stoppage](#stopping-the-watcher), [side effect invalidation](#side-effect-invalidation) (with `onInvalidate` passed to the callback as the 3rd argument instead), [flush timing](#effect-flush-timing) and [debugging](#watcher-debugging).
