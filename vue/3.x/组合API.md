# vue3.x

在2022年2月7日，vue3已经成为新的默认版本，而这一天正好也是春节过后的第一个工作日，也是辞旧迎新，虽说笔者工作中使用的主要技术栈是react，但是也应该对其他前端技术保持关注，所以在这里记录一下自己对于vue3的理解。

## 组合api

vue3与vue2相比其中一个比较大的改动就是组合api了，那么组合api到低有什么好处呢。

### 组合API解决的痛点

首先，我们需要知道一点，任何技术都是为了解决某些问题而诞生的，我们可以回想一下在使用vue2的时候，因为vue2使用的options api给予了我们很多限制，比如说状态只能放在data中，事件只能放在methods中等，这样随着我们的业务日益丰富，我们的组件会变得越来越复杂，这一点是无法避免的，所以如何增加代码的可读性就至关重要，但是vue2的options api并不能很好的解决这一点问题。

设想一下，如果我们的组件有很多的功能，那么按照vue2代码来说，代码结构大概是这样的：

![image](https://v3.cn.vuejs.org/images/options-api.png)

图片中相同颜色的部分代表是同一个功能的代码，我们可以看出同一个功能的代码其实非常分散，假如说我们的某个功能需求发生了更改，需要我们去修改这部分的代码，那么这时候我们就会非常难受，特别是有的时候之前负责这部分的小伙伴离职了我们接手这个项目的时候，因为代码过于分散，从而导致我们需要花费很多的时间才能理解这一部分代码，正是为了解决这个问题，所以才有了组合API。

通过组合API，我们只需要将模版中使用到的数据、方法在setup函数中返回即可

```vue
...

<script>
export default {
  setup() {
    return {
      // 这里返回的东西在模版中都可以使用
    }
  }
}
</script>

...
```

我们在使用组合API的时候没有了data，那么我们该如何去定义我们组件中需要的状态呢，这里vue也是为我们提供了两个新的API， `ref`和`reactive`，我们可以使用这两个API来定义响应式数据，可以看一个例子

```vue
<script setup lang="ts">
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(0)

    const add = () => {
      count.value++;
    }

    return {
      count,
      add,
    }
  }
}
</script>

<template>
  <p>{{ count }}</p>
  <button @click="add">ADD</button>
</template>

...
```

上面代码可以看出，我们使用ref声明了一个响应式的变量count，笔者更喜欢把它叫做状态，另一个API`reactive`多使用与创建引用类型数据，两者的使用方式没有太大区别。

经常使用vue的人可能看得出，我们这里的模版中有两个最外层节点，这种写法在vue3中是不会报错的，在vue3中模版下可以存在多个最外层节点。不过一些人看到这种写法后并没有感觉到这种写法比vue2好多少，因为一堆代码写在setup里setup也会臃肿不堪，并不比vue2好到哪去，但是作为一个react用户来说，明显看到这种写法感觉更好，我们可以通过下面这个案例，来看出组合API的高级使用，而通过下面的案例相信我们也能理解组合API到低是如何解决vue2中的问题。

需求：

  上面一个计数器
  下面一个todoList

由于是案例，这里就不再进行拆分，大家对比vue3和vue2的代码，自行理解。

vue2代码实现：

```vue
<template>
  <div>
    <div class="counter">
      <button @click="minus">minus</button>
      <input :value="count" />
      <button @click="add">add</button>
    </div>

    <div class="todo-list">
      <input v-model="text" />
      <button @click="addItem">add</button>

      <ul>
        <li v-for="item in list" :key="item.id">
          <span>{{ item.content }}</span>
          <button @click="removeItem(item.id)">del</button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        count: 0,
        text: '',
        list: [],
      }
    },
    methods: {
      minus() {
        this.count--;
      },
      add() {
        this.count++;
      },
      addItem() {
        const item = { id: Date.now(), content: this.text };
        this.list.push(item)
        this.text = ""
      },
      removeItem(id) {
        this.list = this.list.filter(item => item.id !== id);
      }
    }
  }
</script>
```

vue3代码实现：

```vue
<script>
import { ref, reactive } from 'vue'

// 计数器逻辑
function useCounter () {
  let count = ref(0);
  const add = () => count.value++;
  const minus = () => count.value--;

  return {
    count,
    add,
    minus,
  }
}

// todolist逻辑
function useTodoList () {
  let text = ref('');
  let list = reactive([]);
  const addItem = () => {
    const item = { id: Date.now(), content: text.value }
    list.push(item)
    text.value = '';
  }
  const removeItem = (index) => {
    list.splice(index, 1)
  }
  return {
    text,
    list,
    addItem,
    removeItem,
  }
}

export default {
  setup() {
    const { count, add, minus } = useCounter();
    const { text, list, addItem, removeItem } = useTodoList();

    return {
      count,
      add,
      minus,
      text, 
      list, 
      addItem, 
      removeItem,
    }
  }
}
</script>

<template>
  <div>
    <div class="counter">
      <button @click="minus">minus</button>
      <input :value="count" />
      <button @click="add">add</button>
    </div>

    <div class="todo-list">
      <input v-model="text" />
      <button @click="addItem">add</button>

      <ul>
        <li v-for="(item, index) in list" :key="item.id">
          <span>{{ item.content }}</span>
          <button @click="removeItem(index)">del</button>
        </li>
      </ul>
    </div>
  </div>
</template>
```

我们可以看到，在使用了vue3的写法之后，我们只需要将逻辑抽离出去，就可以使得setup函数十分清爽，但是setup中需要return的东西太多了，写起来还是不太舒服，因此vue3中也为我们提供了更加方便的预发糖，我们只需要将setup写进script标签中，vue就会把我们在script中定义的状态全部暴露给模版。

```vue
<script setup>
import { ref, reactive } from 'vue'

// 计数器逻辑
function useCounter () {
  let count = ref(0);
  const add = () => count.value++;
  const minus = () => count.value--;

  return {
    count,
    add,
    minus,
  }
}

// todolist逻辑
function useTodoList () {
  let text = ref('');
  let list = reactive([]);
  const addItem = () => {
    const item = { id: Date.now(), content: text.value }
    list.push(item)
    text.value = '';
  }
  const removeItem = (index) => {
    list.splice(index, 1)
  }
  return {
    text,
    list,
    addItem,
    removeItem,
  }
}

const { count, add, minus } = useCounter();
const { text, list, addItem, removeItem } = useTodoList();
</script>

...
```

熟悉react的看到这种写法就会发现，这正是react的自定义hooks啊，两者写法非常类似，但是原理却是完全不一样，我们甚至可以将`useCounter`和`useTodoList`两个自定义hook抽离到一个单独的文件中去

```vue
<script setup>
import { useCounter, useTodoList } from './hooks'

const { count, add, minus } = useCounter();
const { text, list, addItem, removeItem } = useTodoList();
</script>

...
```

而经过这样的逻辑抽离，就算是新接触我们需求的小伙伴，在面对需求发生修改的情况下，比方说新的需求是需要给counter添加最大最小值的限制，那么我们通过组件中的代码可以看出逻辑都在`useCounter`这个hook中，这时我们就无需关注其他地方的代码，只对`useCounter`中对逻辑进行修改即可

```js
// hooks.js
import { ref, reactive } from 'vue'

// 计数器逻辑
function useCounter (options) {
  const { max, min } = options;
  let count = ref(0);
  const add = () => count.value >= max ? (count.value = max) : count.value++;
  const minus = () => count.value <= min ? (count.value = min) : count.value--;

  return {
    count,
    add,
    minus,
  }
}

...
```

组件中

```vue
<script setup>
import { useCounter, useTodoList } from './hooks'

const { count, add, minus } = useCounter({ max: 10, min: 0 });
const { text, list, addItem, removeItem } = useTodoList();
</script>

...
```

这里说了这么多，就是为了体现组合API的好处，使用组合API确实能够增加我们项目代码的可读性，能够有效的降低我们的维护成本，只有代码写的更加规范，我们才能更早的下班。