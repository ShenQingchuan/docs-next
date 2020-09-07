# 介绍

## 为什么需要组合式 API

::: tip 注意
读到这里，我们假设你已经熟悉了[Vue 的基础](introduction.md)和[如何创建组件](component-basics.md)。
:::

创建 Vue 组件使得我们可以将界面中的重复部分，同它的功能一起抽取得到可重用的代码。这将大大增强我们应用的灵活性和可维护性。然而目前的体验证明这也许还不够，特别是当你的应用变得非常大的时候——试想组件数量成百上千的时候。要构建大型应用，复用代码变得尤其重要。

假设在我们的应用中，我们有一个视图来显示某个用户的仓库列表。在此之上，我们希望应用搜索和筛选功能。我们处理这个视图的组件看起来像这样：

```js
// src/components/UserRepositories.vue

export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: { type: String }
  },
  data () {
    return {
      repositories: [], // 1
      filters: { ... }, // 3
      searchQuery: '' // 2
    }
  },
  computed: {
    filteredRepositories () { ... }, // 3
    repositoriesMatchingSearchQuery () { ... }, // 2
  },
  watch: {
    user: 'getUserRepositories' // 1
  },
  methods: {
    getUserRepositories () {
      // using `this.user` to fetch user repositories
    }, // 1
    updateFilters () { ... }, // 3
  },
  mounted () {
    this.getUserRepositories() // 1
  }
}
```

这个组件有几个职责：

1. 通过用户名，调用一个假设的外部 API 获取其仓库列表，并在用户名变化时刷新该列表。
2. 通过一个 `searchQuery` 字符串来搜索
3. 使用 `filters` 对象

大部分情况下我们都是使用组件的选项（`data`, `computed`, `methods`, `watch`）来管理逻辑，然而当我们的组件变得更大时，**逻辑关注点**的数量也在增长。如果不是作者，那么该组件将难以阅读和理解。

![Vue 选项式 API: 代码被组件选项分组](https://user-images.githubusercontent.com/499550/62783021-7ce24400-ba89-11e9-9dd3-36f4f6b1fae2.png)

这个例子展示来一个根据**逻辑关注点**进行染色的大型组件：

这样的碎片分段使得理解和维护一个复杂组件变得困难。选项的分离模糊了基本的逻辑关注点。此外，当处理单个逻辑问题时，我们必须不断地“跳转”相关代码的选项块。

如果我们能够配置与相同逻辑相关的代码，那就更好了。而这正是组合式 API 所能做到的。

## 组合式 API 基础

理解了原因，现在我们来看如何使用。要开始使用组合式 API 我们需要一个能实际使用它的地方。在一个 Vue 组件中，这个地方叫做 `setup`。

### `setup` 组件选项

这个新的 `setup` 组件选项会在组件创建**之前**，当 `props` 被解析后，然后就将作为组合式 API 的入口点。

::: warning 警告
因为当 `setup` 被执行时组件实例还未创建完成，所以在 `setup` 中时无法使用 `this` 的。这意味着，除了 `props` 以外，你将无法访问到其他任何在组件上定义的 property——**组件局部状态**，**计算属性**或**方法**。
:::

`setup` 选项应是一个函数，接受的参数为 `props` 和 [之后](composition-api-setup.html#arguments)会讲到的 `context`。另外，`setup` 返回值中的所有东西（计算属性，方法，生命周期钩子等等）将会暴露该组件的其他选项以及模板。

让我们给组件添加一个 `setup` 选项：

```js
// src/components/UserRepositories.vue

export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: { type: String }
  },
  setup(props) {
    console.log(props) // { user: '' }

    return {} // 这里返回的任何东西都将在组件其他部分可用
  }
  // 组件其他部分
}
```

现在让我们开始抽取第一个逻辑关注点（即之前的标记了 “1” 的代码段）。

> 1. 通过用户名，调用一个假设的外部 API 获取其仓库列表，并在用户名变化时刷新该列表。

我们先从最明显的几个部分开始：

- 仓库列表
- 更新仓库列表的函数
- 返回该列表和该函数使得在组件其他部分可访问

```js
// src/components/UserRepositories.vue `setup` function
import { fetchUserRepositories } from '@/api/repositories'

// 在我们的组件中
setup (props) {
  let repositories = []
  const getUserRepositories = async () => {
    repositories = await fetchUserRepositories(props.user)
  }

  return {
    repositories,
    getUserRepositories // 返回的函数与方法（methods）表现一致
  }
}
```

这只是个开始，意料之中它将不起作用，因为变量 `repositories` 不是响应式的。这意味着从用户角度上看这个仓库列表将始终为空。让我们修复这个问题！

### 使用 `ref` 定义响应式变量

在 Vue 3.0 中我们可以使用一个新的 `ref` 函数使任何变量编委响应式的，就像这样：

```js
import { ref } from 'vue'

const counter = ref(0)
```

`ref` 获取到该参数值后，将它包裹到一个对象中的 `.value` 属性中，对其的访问与更改都将是响应式的，最后返回该对象。

```js
import { ref } from 'vue'

const counter = ref(0)

console.log(counter) // { value: 0 }
console.log(counter.value) // 0

counter.value++
console.log(counter.value) // 1
```

将值包裹进一个对象看上去有些没有必要，但实际上这是为了统一在 JavaScript 中各个数据类型的表现。因为在 JavaScript 中，例如 `Number` 或 `String` 这些基础类型都将以值传递，而非以引用传递。

![传值 vs 传引用](https://blog.penjee.com/wp-content/uploads/2015/02/pass-by-reference-vs-pass-by-value-animation.gif)

又一个包含任意值的包裹对象使得我们能够安全地在整个组件间传递值，而不用担心在某处丢失其响应性。

::: tip 注意
换句话说，`ref` 为值创建了一个**响应式引用**。**引用**的概念将在组合式 API 中十分常见。
:::

回到我们的例子中，让我们创建一个响应式的 `repositories` 变量：

```js
// src/components/UserRepositories.vue `setup` function
import { fetchUserRepositories } from '@/api/repositories'
import { ref } from 'vue'

// 在我们的组件中
setup (props) {
  const repositories = ref([])
  const getUserRepositories = async () => {
    repositories.value = await fetchUserRepositories(props.user)
  }

  return {
    repositories,
    getUserRepositories
  }
}
```

完成了！现在无论在哪里调用 `getUserRepositories`，`repositories` 都将被更改，而视图也会相应地更新。我们的组件将会像这样：

```js
// src/components/UserRepositories.vue
import { fetchUserRepositories } from '@/api/repositories'
import { ref } from 'vue'

export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: { type: String }
  },
  setup (props) {
    const repositories = ref([])
    const getUserRepositories = async () => {
      repositories.value = await fetchUserRepositories(props.user)
    }

    return {
      repositories,
      getUserRepositories
    }
  },
  data () {
    return {
      filters: { ... }, // 3
      searchQuery: '' // 2
    }
  },
  computed: {
    filteredRepositories () { ... }, // 3
    repositoriesMatchingSearchQuery () { ... }, // 2
  },
  watch: {
    user: 'getUserRepositories' // 1
  },
  methods: {
    updateFilters () { ... }, // 3
  },
  mounted () {
    this.getUserRepositories() // 1
  }
}
```

我们已经将第一个逻辑关注点的几个部分移动到了 `setup` 方法中，它们放在了相近的位置，这很棒。我们还剩 `mounted` 钩子中的 `getUserRepositories` 函数调用，再设置一个侦听器监听 `user` 的变化。

我们再从生命周期钩子开始。

### 在 `setup` 中注册生命周期钩子

要使得组合式 API 的功能与选项式 API 完全幂等，我们还需能在 `setup` 中注册生命周期钩子。我们将会使用到 Vue 提供的几个新函数。组合式 API 中的生命周期钩子与选项式 API 有相同但名字但会多一个 `on` 前缀，例如 `mounted` 将会变为 `onMounted`。

这些方法都接收一个回调函数，作为钩子被组件调用执行。

让我们将它们添加到 `setup` 函数中：

```js
// src/components/UserRepositories.vue `setup` function
import { fetchUserRepositories } from '@/api/repositories'
import { ref, onMounted } from 'vue'

// 在我们的组件中
setup (props) {
  const repositories = ref([])
  const getUserRepositories = async () => {
    repositories.value = await fetchUserRepositories(props.user)
  }

  onMounted(getUserRepositories) // 在 `mounted` 阶段会调用 `getUserRepositories`

  return {
    repositories,
    getUserRepositories
  }
}
```

现在我们将对那些 `user` prop 上的变更作出响应。对此我们会使用到一个单独对 `watch` 函数。

### 使用 `watch` 响应变更

就像我们使用 `watch` 选项在组件中设置一个 `user` property 的侦听器那样，我们可以使用从 Vue 引入对 `watch` 函数。它接受三个参数：

- 一个监听对象的**响应式引用**或 getter 函数
- 一个回调函数
- 可选的配置对象

**这里是一个简单示例：**

```js
import { ref, watch } from 'vue'

const counter = ref(0)
watch(counter, (newValue, oldValue) => {
  console.log('计数器新的值是：' + counter.value)
})
```

无论何时 `counter` 被更改，例如 `counter.value = 5`，侦听器会被触发并执行（第二个参数）回调函数，因而会在控制台打印出 `'The new counter value is: 5'`。

**下面是等价的选项式 API 写法：**

```js
export default {
  data() {
    return {
      counter: 0
    }
  },
  watch: {
    counter(newValue, oldValue) {
      console.log('计数器新的值是：' + this.counter)
    }
  }
}
```

关于 `watch` 的更多细节，请参考 [深入了解](TODO).

**让我们将其应用到之前的示例中：**

```js
// src/components/UserRepositories.vue `setup` function
import { fetchUserRepositories } from '@/api/repositories'
import { ref, onMounted, watch, toRefs } from 'vue'

// 在我们的组件中
setup (props) {
  // 使用 `toRefs` 将 props 中的 `user` 属性转为响应式引用。
  const { user } = toRefs(props)

  const repositories = ref([])
  const getUserRepositories = async () => {
    // 通过 `user.value` 访问到引用值，来更新 `props.user`
    repositories.value = await fetchUserRepositories(user.value)
  }

  onMounted(getUserRepositories)

  // 对 user prop 的响应式引用设置一个侦听器
  watch(user, getUserRepositories)

  return {
    repositories,
    getUserRepositories
  }
}
```

你可能注意到上面例子里在 `setup` 中使用到了 `toRefs`。这是为了确保侦听器能够对 `user` prop 的变更作出响应。

通过上述更改，我们已经将第一个逻辑关注点移动到了同一处，对第二个关注点也是一样。——根据 `searchQuery` 过滤，这一次我们将用到一个计算属性。

### 单独的 `computed` 计算属性

与 `ref` 和 `watch` 类似，计算属性同样可以通过从 Vue 引入的 `computed` 函数，在一个 Vue 组件外被创建。让我们回到计数器的例子中：

```js
import { ref, computed } from 'vue'

const counter = ref(0)
const twiceTheCounter = computed(() => counter.value * 2)

counter.value++
console.log(counter.value) // 1
console.log(twiceTheCounter.value) // 2
```

在这里，`computed` 函数通过第一个参数，一个 getter 的回调函数返回了一个 _只读的_ **响应式引用**。为了访问到新创建的计算属性变量的**值**，我们需要使用 `.value`，就像 `ref` 那样。

让我们将搜索功能移动到 `setup` 函数中：

```js
// src/components/UserRepositories.vue `setup` function
import { fetchUserRepositories } from '@/api/repositories'
import { ref, onMounted, watch, toRefs, computed } from 'vue'

// 在我们的组件中
setup (props) {
  // 使用 `toRefs` 将 props 中的 `user` 属性转为响应式引用。
  const { user } = toRefs(props)

  const repositories = ref([])
  const getUserRepositories = async () => {
    // 通过 `user.value` 访问到引用值，来更新 `props.user`
    repositories.value = await fetchUserRepositories(user.value)
  }

  onMounted(getUserRepositories)

  // 对 user prop 的响应式引用设置一个侦听器
  watch(user, getUserRepositories)

  const searchQuery = ref('')
  const repositoriesMatchingSearchQuery = computed(() => {
    return repositories.value.filter(
      repository => repository.name.includes(searchQuery.value)
    )
  })

  return {
    repositories,
    getUserRepositories,
    searchQuery,
    repositoriesMatchingSearchQuery
  }
}
```

我们对其他**逻辑关注点**也进行同样的操作，但你可能已经在思考了——_将所有代码都移动到 `setup` 选项中，它不就变得非常大了么？_ 答案是肯定的。所以我们接下来将带来更多原则，我们将首先把上述的代码抽取到单独的**组合函数**中，创建一个 `useUserRepositories`：

```js
// src/composables/useUserRepositories.js

import { fetchUserRepositories } from '@/api/repositories'
import { ref, onMounted, watch } from 'vue'

export default function useUserRepositories(user) {
  const repositories = ref([])
  const getUserRepositories = async () => {
    repositories.value = await fetchUserRepositories(user.value)
  }

  onMounted(getUserRepositories)
  watch(user, getUserRepositories)

  return {
    repositories,
    getUserRepositories
  }
}
```

接下来是搜索的功能：

```js
// src/composables/useRepositoryNameSearch.js

import { ref, computed } from 'vue'

export default function useRepositoryNameSearch(repositories) {
  const searchQuery = ref('')
  const repositoriesMatchingSearchQuery = computed(() => {
    return repositories.value.filter(repository => {
      return repository.name.includes(searchQuery.value)
    })
  })

  return {
    searchQuery,
    repositoriesMatchingSearchQuery
  }
}
```

**现在有了上面两个分列在两个文件中的功能之后，在组件中使用它们。最后完成如下：**

```js
// src/components/UserRepositories.vue
import useUserRepositories from '@/composables/useUserRepositories'
import useRepositoryNameSearch from '@/composables/useRepositoryNameSearch'
import { toRefs } from 'vue'

export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: { type: String }
  },
  setup (props) {
    const { user } = toRefs(props)

    const { repositories, getUserRepositories } = useUserRepositories(user)

    const {
      searchQuery,
      repositoriesMatchingSearchQuery
    } = useRepositoryNameSearch(repositories)

    return {
      // 因为我们并不关注过滤掉的仓库
      // 通过 `repositories` 我们只将过滤出的仓库暴露出去
      repositories: repositoriesMatchingSearchQuery,
      getUserRepositories,
      searchQuery,
    }
  },
  data () {
    return {
      filters: { ... }, // 3
    }
  },
  computed: {
    filteredRepositories () { ... }, // 3
  },
  methods: {
    updateFilters () { ... }, // 3
  }
}
```

看到这里你或许已经懂了，因此让我们跳到最后，迁移剩余的过滤功能。我们不太需要深入实现细节，因为这不是本指南的重点。

```js
// src/components/UserRepositories.vue
import { toRefs } from 'vue'
import useUserRepositories from '@/composables/useUserRepositories'
import useRepositoryNameSearch from '@/composables/useRepositoryNameSearch'
import useRepositoryFilters from '@/composables/useRepositoryFilters'

export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: { type: String }
  },
  setup(props) {
    const { user } = toRefs(props)

    const { repositories, getUserRepositories } = useUserRepositories(user)

    const {
      searchQuery,
      repositoriesMatchingSearchQuery
    } = useRepositoryNameSearch(repositories)

    const {
      filters,
      updateFilters,
      filteredRepositories
    } = useRepositoryFilters(repositoriesMatchingSearchQuery)

    return {
      // 因为我们并不关注过滤掉的仓库
      // 通过 `repositories` 我们只将过滤出的仓库暴露出去
      repositories: filteredRepositories,
      getUserRepositories,
      searchQuery,
      filters,
      updateFilters
    }
  }
}
```

到这里示例就完成了！

目前我们只是触及了组合式 API 的表层，以及它允许我们做什么。要了解更多信息，请参阅深入指南。

<!-- TODO：等待官方的深入指南 -->
