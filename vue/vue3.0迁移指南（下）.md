> VUE3.0最大的更新就是加入了compositionAPI，我相信你在阅读完这文章后，对3.0的用法会有更深入的认识。
>
> 如果你想更好地过渡到3.0，建议看一下上篇文章

### **为什么要引入一个全新的API？**

在 V3.0 以前，我们开发一个组件都使用一个固定的模板结构，我相信大多数使用VScode开发的小伙伴们，已经把该模板配置到了vscode中。当然这个模板你可以自定义为你常用的内容和结构。

```vue
<template>
  <div class="wrapper"></div>
</template>

<script>
export default {
  components:{},
  props:{},
  data(){
    return {
    }
  },
  watch:{},
  computed:{},
  methods:{},
  created(){},
  mounted(){}
}
</script>
<style lang="stylus" scoped>
.wrapper{}
</style>
```

​	发过程也很简单，在 `data` 中存放数据，`props` 中接受父组件的传值，不同生命周期函数中处理要做的事，`watch` 中监听数据的变化等等。通过 `this` 可以直接获取到 `data` 、`props` 、`methods` 等的值或调用方法。

​	任何软件、平台、操作系统等等都会存在更新迭代的过程，每次更新迭代所带来的新鲜事物都是有它的道理的，目的都是为了提成产品的质量。Vue这次大变革也是让vue有了更多的用武之地。

​	VUE简单易学，在人们的认知里面更适用于构建中小型项目，React在大型项目上更上一筹。但随着vue的影响力日益扩大，许多用户也开始使用Vue构建更大型的项目，随着项目的日益迭代，在原来的基础上不断的维护和新增内容，慢慢的可以感受到vue当前 API 所带来的的编程模型的限制。主要体现可以分为两个：

- 随着功能的不断增加，复杂组件的代码变得越来越难以阅读和理解。
- 目前缺少一种简洁且低成本的机制来提取和重用多个组件之间的逻辑

#### 

### 使用方法

​	说实话，我并不知道这个讲解怎么开头，在网上也可以找到很多compositionAPI的使用方法，我自己还并没有真正的使用它去开发项目，我希望可以以更加通俗易懂的方式去学习如何使用它。

​	让我们从一段代码开始：

```vue
<template>
  <div>{{ count }} {{ object.foo }}</div>
</template>
<script>
    import { ref, reactive } from 'vue'
    export default {
      props: {
        name: String
      },
      setup(props, context) {
        console.log(props);
        console.log(context);
		const count = ref(0);
        const object = reactive({
            foo: 'bar'
        });
        return {
			count,
            object
        }
      }
    }
</script>
```

​	`setup` 函数是一个新的组件选项。作为在组件内使用 Composition API 的入口点，它会在创建组件实例->初始化 `props` 后立即调用，在 `beforeCreate` 生命周期前调用。

​	如果 `setup` 返回的一个对象，对象中的属性将会合并到组件模板的渲染上下文中，也就是说你可以在 `<template>` 中直接使用对象中的值，就像上面的例子一样。

​	`setup` 也可以返回一个函数，在函数中也可以使用当前`setup` 函数作用域中的响应式数据：

```javascript
import { h, ref, reactive } from 'vue'

export default {
  setup() {
    const count = ref(0)
    const object = reactive({ foo: 'bar' })

    return () => h('div', [count.value, object.foo])
  },
}
```

#### `setup` 的参数

`setup` 接收两个参数，第一个参数是父组件所传递的值，也就是 `props` ，第二个参数提供了上下文对象。

- `props` 

  这里的 `props` 是具有响应式的，它可以被  `watchEffect` 或 `watch` 侦听：

```javascript
export default {
  props: {
    name: String,
  },
  setup(props) {
    watchEffect(() => {
      console.log(`name is: ` + props.name)
    })
  },
}
```

​	**注**：不要尝试对 `props` 进行结构操作，这会使其失去响应性：

```javascript
export default {
  props: {
    name: String
  },
  setup(props) {
    const { name } = props;
    console.log(name);
    return {
    }
  }
}
```

​	你可以使用 `eslint` 来检测你的代码，在你进行结构操作时发出警告：

![](https://www.dengzhanyong.com/PHP/images/1603187312.png)

##### 第二个参数 `context`

​	在讲 `context` 之前我必须要告诉你，在 `setup` 中 `this` 不再可用，这与2.x中的 `this` 完全不同；

​	在2.x中 ，`this ` 可以获取到 `data` 中的内容。许多Vue的插件都向 `this` 中注入 property。例如 Vue Router 注入 `this.$route` 和 `this.$router`，而 Vuex 注入 `this.$store`。这使得类型推导变得很有技巧性，因为每个插件都要求用户为注入的 property 向 Vue 增加类型定义。

​	但是在 `composition API` 中，你不能再通过 `this` 这么做，在 `setup ` 中 `this` 的值是 `undefined` 。取而代之的是，在插件内部利用 `provide` 和 `inject` 并暴露一个组合函数。

```javascript
const StoreSymbol = Symbol()

export function provideStore(store) {
  provide(StoreSymbol, store)
}

export function useStore() {
  const store = inject(StoreSymbol)
  if (!store) {
    // 抛出错误，不提供 store
  }
  return store
}
```

​	没有了 `this` ，子组件如何向父组件通过发送事件来通信呢？

```javascript
export default {
  setup(props, context) {
    console.log(context);
    return {
    }
  }
}
```

​	看一下打印结果：

![](https://www.dengzhanyong.com/PHP/images/1603188188.png)

​		`attrs` 和 `slots` 都是内部组件实例上对应的代理，可以确保在更新后仍然是最新的值。所以可以解构，无需担心它的值会过期：

```javascript
const MyComponent = {
  setup(props, { attrs }) {
    // 一个可能之后回调用的签名
    function onClick() {
      console.log(attrs.foo) // 一定是最新的引用，没有丢失响应性
    }
  },
}
```

​	可以通过 `emit` 向父组件通过事件的形式传值，使用方法和原来一样，只是写法上有些改变：

```javascript
// 2.x中
this.$emit('eventName', value);

// composition Api 中
export default {
  setup(props, { emit }) {
    function handleClick(value) {
        emit('eventName', value);
    }
    return {
        handleClick
    }
  }
}
```

##### 为什么要把 `props` 作为第一个参数，而不是包含在上下文中：

- 组件内使用 `props` 的场景更多，基本上每个组件都会使用到，甚至是只使用 `props`
- 可以让 TypeScript 对 `props` 单独做类型推导，不会和上下文中的其他属性相混淆。



#### 生命周期钩子函数

​	在 `composition API` 中，生命周期钩子函数也被移到了 `setup` 中，移除了 `beforeCreate` 和 `created` 生命周期，这里面的内容都可以在 `setup` 中编写。其他的生命周期在名称上都多了一个前缀 `on` 。

| 2.x              | composition Api |
| ---------------- | --------------- |
| ~~beforeCreate~~ | setup()         |
| ~~created~~      | setup()         |
| beforeMount      | onBeforeMount   |
| mounted          | onMounted       |
| beforeUpdate     | onBeforeUpdate  |
| updated          | onUpdated       |
| beforeDestroy    | onBeforeUnmount |
| destroyed        | onUnmounted     |
| errorCaptured    | onErrorCaptured |

​	同时还增加了两个用于调试的钩子函数：

- `onRenderTracked`
- `onRenderTriggered`

  他们都接收一个参数 `DebuggerEvent` ，用于检查是哪个依赖性导致组件的重新渲染。

 所有的生命周期钩子函数在使用前必须要引入，并且他们只能在 `setup` 中使用：

```javascript
import { onMounted, onUpdated, onUnmounted } from 'vue'

const MyComponent = {
  setup() {
    onMounted(() => {
      console.log('mounted!')
    })
    onUpdated(() => {
      console.log('updated!')
    })
    onUnmounted(() => {
      console.log('unmounted!')
    })
  },
}
```

#### 响应式属性

​	在2.x中，定义在 `data ` 方法返回的对象中所有的属性都具有响应式的特性，当改变值后相应的界面也会自动更新。

​	在3.0中，有两个API可以获得响应式属性。

- **`reactive`**

它接收一个普通对象作为参数，返回该普通对象的响应式代理，也就是所有Vue用户都应该熟悉的响应式对象。

```javascript
import { reactive } from 'vue'
export default {
  setup() {
    // state 现在是一个响应式的状态
    const state = reactive({
      count: 0,
    })
    return {
        state
    }
  }
}
```

在模板中你可以使用 `{{state.count}}` 来显示对应的内容。

如果你在 `setup` 的返回值中对 `state` 进行了解构，请不要这么做，这会导致它失去响应式的特性。

- **`ref`**

接受一个参数值并返回一个响应式且可改变的 ref 对象。ref 对象拥有一个指向内部值的单一属性 `.value`。

```javascript
const count = ref(0)
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```

如果 `ref` 的返回值在 `setep` 的返回值中时，你不需要加上 `.value` ，它会自动解套：

```vue
<template>
  <div>{{ count }}</div>
</template>

<script>
  export default {
    setup() {
      return {
        count: ref(0),
      }
    },
  }
</script>
```

你若将 `ref` 作为 `reactive` 对象的属性访问或修改时，也将会自动解套，其行为类似普通属性：

```javascript
const count = ref(0)
const state = reactive({
  count,
})

console.log(state.count) // 0

state.count = 1
console.log(count.value) // 1
```

注意如果将一个新的 ref 分配给现有的 ref， 将替换旧的 ref：

```javascript
const otherCount = ref(2)

state.count = otherCount
console.log(state.count) // 2
console.log(count.value) // 1
```

**注**：只有嵌套在 `reactive` 对象中时， `ref` 才会自动解套。从 `Array` 或者 `Map` 等原生集合类中访问 ref 时，不会自动解套：

```javascript
const arr = reactive([ref(0)])
// 这里需要 .value
console.log(arr[0].value)

const map = reactive(new Map([['foo', ref(0)]]))
// 这里需要 .value
console.log(map.get('foo').value)
```

 `ref` 的参数也可以一个对象，此时它将会调用 `reactive` 方法进行深层次转换。

##### 模板Refs

当使用组合式 API 时，*reactive refs* 和 *template refs* 的概念已经是统一的。为了获得对模板内元素或组件实例的引用，我们可以像往常一样在 `setup()` 中声明一个 ref 并返回它：

```vue
<template>
  <div ref="root"></div>
</template>

<script>
  import { ref, onMounted } from 'vue'

  export default {
    setup() {
      const root = ref(null)

      onMounted(() => {
        // 在渲染完成后, 这个 div DOM 会被赋值给 root ref 对象
        console.log(root.value) // <div/>
      })

      return {
        root,
      }
    },
  }
</script>
```

**在 `v-for` 中使用**

模板 ref 在 `v-for` 中使用 vue 没有做特殊处理，需要使用**函数型的 ref**（3.0 提供的新功能）来自定义处理方式：

```vue
<template>
  <div v-for="(item, i) in list" :ref="el => { divs[i] = el }">
    {{ item }}
  </div>
</template>

<script>
  import { ref, reactive, onBeforeUpdate } from 'vue'

  export default {
    setup() {
      const list = reactive([1, 2, 3])
      const divs = ref([])

      // 确保在每次变更之前重置引用
      onBeforeUpdate(() => {
        divs.value = []
      })

      return {
        list,
        divs,
      =}
    },
  }
</script>
```

##### Ref vs. Reactive

`ref` 和 `reactive` 都可以定义响应式属性，使用 `ref` 在修改时需要加上 `.value` ，使用 `reactive` 不能解构。你肯定会纠结到底使用哪一个更好。

他们各自肯定都有所存在的意义，只有对他们有更深入的了解才能更高效的使用他们。 `ref` 与 `reactive` 就像是对应下面两种风格的写法：

```javascript
// 风格 1: 将变量分离
let x = 0
let y = 0

function updatePosition(e) {
  x = e.pageX
  y = e.pageY
}

// --- 与下面的相比较 ---

// 风格 2: 单个对象
const pos = {
  x: 0,
  y: 0,
}

function updatePosition(e) {
  pos.x = e.pageX
  pos.y = e.pageY
}
```

##### 解决 `reactive` 不能解构的办法

 可以使用 `toRefs` 把响应式对象的每个 property 都转成相应的 `ref `:

```javascript
function useMousePosition() {
  const pos = reactive({
    x: 0,
    y: 0,
  })

  // ...
  return toRefs(pos)
}
```

你也可以使用 ``toRef`` 只对某个属性就行操作，这个被创建的 `ref` 可以被传递并且能够保持响应式：

```javascript
const state = reactive({
  foo: 1,
  bar: 2,
})

const fooRef = toRef(state, 'foo')

fooRef.value++
console.log(state.foo) // 2

state.foo++
console.log(fooRef.value) // 3
```

#### 计算属性

传入一个 getter 函数，返回一个默认不可手动修改的 ref 对象。

```javascript
import { reactive, computed } from 'vue'

const state = reactive({
  count: 0,
})

const double = computed(() => state.count * 2)
```

`computed` 是如何实现的呢？我们可以猜想一下，可能像是下面这样：

```javascript
function computed(getter) {
  let value
  watchEffect(() => {
    value = getter()
  })
  return value
}
```

但是这样做会有一个问题，如果 `value` 是一个值类型的基础类型，一个响应式的值一旦作为 property 被赋值或从一个函数中返回后，那么得到的结果会失去响应式。如果是引用类型的话那么就是正常的：

![](https://www.dengzhanyong.com/PHP/images/1603265972.gif)

对代码进行一些改进：

```javascript
function computed(getter) {
  const ref = {
    value: null,
  }
  watchEffect(() => {
    ref.value = getter()
  })
  return ref
}
```

当然，这样做也是需要付出代价的，我们在使用时就需要在后面加上 `.value` 。不过我们可以每次都获取到最新的值。

#### `readonly`

它接收一个对象（响应式或普通）或 ref作为参数，返回该对象的**只读**代理，并且是“深层的”，也就是说对象内部的任何嵌套属性也都是**只读**的。

```javascript
const original = reactive({ count: 0 })

const copy = readonly(original)

watchEffect(() => {
  // 依赖追踪
  console.log(copy.count)
})

// original 上的修改会触发 copy 上的侦听
original.count++

// 无法修改 copy 并会被警告
copy.count++ // warning!
```

#### `watchEffect`

立即执行传入的一个函数，并响应式追踪其依赖，并在其依赖变更时重新运行该函数。

```javascript
const count = ref(0)

watchEffect(() => console.log(count.value))
// -> 打印出 0

setTimeout(() => {
  count.value++
  // -> 打印出 1
}, 100)
```

当 `watchEffect` 在组件的 `setup()` 函数或生命周期钩子被调用时， 侦听器会被链接到该组件的生命周期，并在组件卸载时自动停止。

#### `watch`

`watch` 的行为与2.x中的 `watch` 相同，用来监听数据源的变化：

```javascript
import { ref, watch } from 'vue'

export default {
  const count = ref(0)
  watch(count, (newVal, oldVal) => {
    console.log(newVal, oldVal);
  })
  return {
      count
  }
}
```

如果想侦听多个数据源，可以把所有被侦听的数据组合成一个数组作为第一个参数传入，相应的在方法中进行结构：

```javascript
watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {
  /* ... */
})
```

#### 其他API

##### `isRef`

检查一个值是否为一个 ref 对象。

##### `isProxy`

检查一个对象是否是由 `reactive` 或者 `readonly` 方法创建的代理。

##### `isReactive`

检查一个对象是否是由 `reactive` 创建的响应式代理。

如果这个代理是由 `readonly` 创建的，但是又被 `reactive` 创建的另一个代理包裹了一层，那么同样也会返回 `true`。

##### `isReadonly`

检查一个对象是否是由 `readonly` 创建的只读代理。



#### 高级响应式系统API

除了上面那些常用的API外，还有一些高级的API，我自己还并没有去深入的学习使用，因此在这里我只会把他们罗列出来：

`customRef` 、 `markRaw` 、  `shallowReactive`、  `shallowReadonly`、  `shallowRef`、  `toRaw` 







​	

