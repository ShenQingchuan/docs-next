# 生命周期钩子

> 这篇假设你已经阅读过了 [组合式 API](composition-api-introduction.html) 和 [响应式原理](reactivity-fundamentals.html)，如果你还对组合式 API 不太了解，推荐你先阅读它。

你可以通过一个以 `on` 为前缀的函数来访问对应的生命周期钩子。

下面这张表罗列出了生命周期钩子在 [`setup()`](composition-api-setup.html) 中是如何调用的：

| 选项式 API        | `setup` 函数中的钩子函数 |
| ----------------- | ------------------------ |
| `beforeCreate`    | 不需要 \*                |
| `created`         | 不需要 \*                |
| `beforeMount`     | `onBeforeMount`          |
| `mounted`         | `onMounted`              |
| `beforeUpdate`    | `onBeforeUpdate`         |
| `updated`         | `onUpdated`              |
| `beforeUnmount`   | `onBeforeUnmount`        |
| `unmounted`       | `onUnmounted`            |
| `errorCaptured`   | `onErrorCaptured`        |
| `renderTracked`   | `onRenderTracked`        |
| `renderTriggered` | `onRenderTriggered`      |

:::tip 注意
因为 `setup` 是在 `beforeCreate` 和 `created` 两个生命周期钩子之间执行，你不需要显式定义出他们，换句话说，你想在这些钩子里写的任何代码都应该直接写在 `setup` 函数里。
:::

这些函数接受一个回调，当该钩子被组件调用时执行该回调将。

```js
// MyBook.vue

export default {
  setup() {
    // mounted
    onMounted(() => {
      console.log('Component is mounted!')
    })
  }
}
```
