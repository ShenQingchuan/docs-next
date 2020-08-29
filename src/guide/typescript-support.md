# TypeScript 支持

> [Vue CLI](https://cli.vuejs.org) 提供了内建的 TypeScript 工具支持。

## 发布为 NPM 包的官方声明文件

静态类型系统能帮助你有效防止许多潜在的运行时错误，而且随着你的应用日渐丰满会更加显著。这也是为什么 Vue 3 采用 TypeScript 编写。这意味着在 Vue 中你不需要任何额外的工具来使用 TypeScript——我们将其作为首要支持目标。

## 推荐配置

```js
// tsconfig.json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    // 这项使得对 this 上的 data property 进行更严格的推断
    "strict": true,
    "moduleResolution": "node"
  }
}
```

注意你需要引入 `strict: true` (或者至少 `noImplicitThis: true`，这是 `strict` 模式的一部分) 以利用组件方法中 `this` 的类型检查，否则它会始终被看作 any 类型。

查看 [TypeScript 编译器选项文档](https://www.typescriptlang.org/docs/handbook/compiler-options.html)了解更多细节。

## 开发工具链

### 工程创建

[Vue CLI](https://github.com/vuejs/vue-cli) 可以使用 TypeScript 生成新工程。创建方式：

```bash
# 1. 如果没有安装 Vue CLI 就先安装
npm install --global @vue/cli@next

# 2. 创建一个新工程，并选择 "Manually select features (手动选择特性)" 选项
vue create my-project-name

# 如果你已经有了一个以 Vue CLI 创建，但没有包含 TypeScript 的项目，请添加一个相应的 Vue CLI 插件：
vue add typescript
```

确保单文件组件当中的 `script` 部分上标注使用语言为 TypeScript。

```html
<script lang="ts">
  ...
</script>
```

### 编辑器支持

要使用 TypeScript 开发 Vue 应用程序，我们强烈建议您使用 [Visual Studio Code](https://code.visualstudio.com/)，它为 TypeScript 提供了极好的“开箱即用”支持。如果你正在使用[单文件组件](./single-file-components.html) (SFC)，可以安装提供 SFC 支持以及其他更多实用功能的 [Vetur 插件](https://github.com/vuejs/vetur)。

[WebStorm](https://www.jetbrains.com/webstorm/) 同样为 TypeScript 和 Vue 提供了“开箱即用”的支持。

## 定义 Vue 组件

为了能让 TypeScript 准确推断 Vue 组件选项中的类型，你需要使用全局方法 `defineComponent` 来定义组件：

```ts
import { defineComponent } from 'vue'

const Component = defineComponent({
  // 类型推断在此可用
})
```

## 与选项式 API 配合使用

TypeScript 应该无需显示定义类型就能够推断大多数的类型。例如，如果你有一个含数字类型 `counter` property 的组件，当你试图对它使用一个字符串独有方法时会报错：

```ts
const Component = defineComponent({
  data() {
    return {
      count: 0
    }
  },
  mounted() {
    const result = this.count.split('') // => 将报错：Property 'split' does not exist on type 'number'
  }
})
```

如果你有一个复杂的类型或接口，你可以使用 [类型断言](https://www.typescriptlang.org/docs/handbook/basic-types.html#type-assertions)来指出其类型：

```ts
interface Book {
  title: string
  author: string
  year: number
}

const Component = defineComponent({
  data() {
    return {
      book: {
        title: 'Vue 3 Guide',
        author: 'Vue Team',
        year: 2020
      } as Book
    }
  }
})
```

### 标注返回类型

由于 Vue 声明文件的循环特性，TypeScript 可能难以推断计算属性的类型。由于这个原因，您可能需要将其显示标注出来：

```ts
import { defineComponent } from 'vue'

const Component = defineComponent({
  data() {
    return {
      message: 'Hello!'
    }
  },
  computed: {
    // 需要标注
    greeting(): string {
      return this.message + '!'
    }

    // 在含 setter 的计算属性中，getter 需要标注返回类型
    greetingUppercased: {
      get(): string {
        return this.greeting.toUpperCase();
      },
      set(newValue: string) {
        this.message = newValue.toUpperCase();
      },
    },
  }
})
```

### 标注 Prop

Vue 将会通过一个预定义的 `type` 做运行时校验。若要将这些类型提供给 TypeScript，我们需要通过 `PropType` 来转换这些构造器：

```ts
import { defineComponent, PropType } from 'vue'

interface ComplexMessage {
  title: string
  okMessage: string
  cancelMessage: string
}
const Component = defineComponent({
  props: {
    name: String,
    success: { type: String },
    callback: {
      type: Function as PropType<() => void>
    },
    message: {
      type: Object as PropType<ComplexMessage>,
      required: true,
      validator(message: ComplexMessage) {
        return !!message.title
      }
    }
  }
})
```

如果你发现验证器并没有得到类型推导或者没有成员变量的补全提示，那么请标注出参数的期望类型。

## 与组合式 API 配合使用

在 `setup()` 函数中，你不需要为 `props` 参数传递类型，它将直接从组件的 `props` 选项中自动推断得到。

```ts
import { defineComponent } from 'vue'

const Component = defineComponent({
  props: {
    message: {
      type: String,
      required: true
    }
  },

  setup(props) {
    const result = props.message.split('') // 正确，'message' 是字符串类型
    const filtered = props.message.filter(p => p.value) // an error will be thrown: Property 'filter' does not exist on type 'string'
  }
})
```

### `ref` 的类型

Ref 是通过传入的初始值来推断类型的：

```ts
import { defineComponent, ref } from 'vue'

const Component = defineComponent({
  setup() {
    const year = ref(2020)

    const result = year.value.split('') // => Property 'split' does not exist on type 'number'
  }
})
```

有时我们可能需要明确指出 ref 内部值的复杂类型，只需在调用 ref 时传入一个泛型参数覆盖默认推导即可。

```ts
const year = ref<string | number>('2020') // year 的类型: Ref<string | number>

year.value = 2020 // 正确！
```

::: tip 注意
如果泛型未知，建议将 `ref` 转换为 `Ref<T>`。
:::

### `reactive` 的类型

当要检查 `reactive` property 的类型时可以使用接口：

```ts
import { defineComponent, reactive } from 'vue'

interface Book {
  title: string
  year?: number
}

export default defineComponent({
  name: 'HelloWorld',
  setup() {
    const book = reactive<Book>({ title: 'Vue 3 Guide' })
    // 或者
    const book: Book = reactive({ title: 'Vue 3 Guide' })
    // 或者
    const book = reactive({ title: 'Vue 3 Guide' }) as Book
  }
})
```

### `computed` 的类型

计算属性会自动从内部返回值中推断类型：

```ts
import { defineComponent, ref, computed } from 'vue'

export default defineComponent({
  name: 'CounterButton',
  setup() {
    let count = ref(0)

    // 只读的
    const doubleCount = computed(() => count.value * 2)

    const result = doubleCount.value.split('') // => 报错：Property 'split' does not exist on type 'number'
  }
})
```
