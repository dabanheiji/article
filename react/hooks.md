[toc]

# hooks的使用心得

`hooks`是`react16.8`版本推出的新特性，`hooks`增强了函数组件的功能，使得函数组件的功能可以媲美类组件，甚至在大多场景下的表现比类组件更加出色，而**函数组件+hooks**的开发模式也是现在的主流开发模式。

## 内置hooks

`react`中内置了一些便捷的`hooks`极大程度的便捷了我们的开发，使用`hooks`的时候需要注意:

1. `hooks`只能在`react`函数中使用
2. `hooks`是有顺序的，所以只能在函数内部的最顶层使用，不能在`if`和`for`循环中使用

这些限制是因为`hooks`是通过链表去记录每个`hooks`对应的顺序状态，如果在循环或者判断中去使用会导致每次渲染的时候`hooks`的执行顺序会与链表中储存的顺序出现不一致从而导致部分`hooks`失效。

下面是一些笔者在日常开发中经常用到的一些`hooks`

### uaeState

`useState`是最简单的`hooks`，相信也是绝大数开发者接触的第一个`hooks`，它的作用是使得函数组件也能够拥有自己的状态。

语法：
```jsx
const [state, setState] = useState(initialValue)
```

`useState`接收一个任意类型的参数作为这个`hooks`的初始值，返回一个长度为2的数组，数组的第一项是我们定义的状态值，第二项是修改状态的方法。

🌰：

```jsx
import React, { useState } from 'react'

function App () {

  const [count, setCount] = useState(1)

  return (
    <div>
      <h1>{ count }</h1>
      <button onClick={() => setCount(count - 1)}>-</button>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  )
}

export default App
```

上面这个计数器的`demo`中可以知道`useState`的基本用法，不过在使用的时候还是有一些注意事项，`useState`返回的第二个方法除了传入新的状态值之外还可以传入一个函数，此函数的参数是当前的`state`值，比如

```jsx
const [count, setCount] = useState(1)

setCount(x => x + 1) // 效果大多时候等同于 setCount(count + 1)
```

这种方式常用于我们修改的新值需要通过旧值计算的情况下。

🌰：

```jsx
...

function App(){
  const [count, setCount] = useState(1)

  const oldAdd = () => {
    setCount(count + 1)
    setCount(count + 1)
    setCount(count + 1)
    // count只会加1
  }

  const newAdd = () => {
    setCount(x => x + 1)
    setCount(x => x + 1)
    setCount(x => x + 1)
    // count会加3
  }

  return (
    <div>
      <h1>{ count }</h1>
      <button>add3</button>
      <button>add3 * 1</button>
    </div>
  )
}

...
```

因为`react`内部做了一些优化为了避免频繁的更新`state`所以会在整个函数执行后去做一次总的更新，所以在`oldAdd`方法中所写的三个`count`对应的值都是一样的，第一次点击的时候相当于执行了三次`setCount(2)`所以只会增加1，而第二中写法因为是一个函数，每次计算都会依据前一个`count`去计算所以相当于

```jsx
1 => 1 + 1 // 2
2 => 2 + 1 // 3
3 => 3 + 1 // 4
```

故而最终会计算出4，需要注意在计时器中我们需要使用第二中写法。另外在`state`为引用类型的时候需要注意，如果设置的新state与旧的state内存地址一样页面是不会被重新渲染的

```jsx
const [state, setState] = useState({
  name: 'Jack',
  age: 18
})

// 不会重新渲染❌
setState(state => {
  state.name = 'Tom'
  return state
})

// 会重新渲染✅
setState(state => ({ ...state, name: 'Tom' }))
```

### useEffect

`useEffect`也是非常常用的一个`hooks`，这个`hooks`我们常用来模拟生命周期。

语法：
```jsx
useEffect(effect, [deps])
```

它接收两个参数，第一个参数可以是是一个有副作用的函数，比如发送请求，dom操作，定时器等，这个函数会在组件渲染之后执行，第二个参数是它的依赖项是可传的，依赖项必须是数组类型，如果不传的话它将在每次渲染之后执行，大多时候我们需要传入依赖项对其做一些限制。

🌰：

```jsx
import React, { useState, useEffect } from 'react'

// 模拟ajax请求
function getData(){
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve([
        { name: 'Jack' },
        { name: 'Tom' },
        { name: 'Rose' },
      ])
    }, 1000)
  })
}

function App(){

  const [list, setList] = useState([])

  useEffect(() => {
    initList()
  }, [])

  const initList = async () => {
    const res = await getData();
    setList(res)
  }

  return (
    <div>
      <ul>
        {
          list.map(item => (
            <li key={item.name}>{ item.name }</li>
          ))
        }
      </ul>
    </div>
  )
}

export default App;
```

这个🌰中我们在`useEffect`中发送了一个请求获取数据，它的依赖项是空的，所以只会在组件首次渲染的时候执行一次，这时候就起到了类组件中`componentDidMount`的作用。当然，我们有时候在`useEffect`中处理的并不一定是请求，而是定义定时器，那么我们在组件卸载的时候就希望移除定时器，这个时候可以在`useEffect`的第一个参数函数中返回一个清除副作用的函数，返回的这个函数会在这个effect移除时执行，假设我们现在有一个时钟组件。

🌰：

```jsx
import React, { useState, useEffect } from 'react'
import moment from 'moment'

const TIME_FORMAT = "YYYY-MM-DD HH:mm:ss";
const getNow = () => moment().format(TIME_FORMAT)
let timer

function App () {

  const [time, setTime] = useState(getNow())

  useEffect(() => {
    // 组件渲染后开启定时器
    timer = setInterval(() => {
      setTime(getNow())
    }, 1000)

    return () => {
      // 组件卸载的时候执行卸载定时器
      timer && clearInterval(timer)
    }
  }, [])

  return (
    <h1>{ time }</h1>
  )
}

export default App
```

此案例中因为依赖项是空数组，所以它会在首次渲染的时候开启定时器，在组件移除时清除定时器，相当于类组件中的`componentWillUnmount`，还有些场景，我们希望用户在选择类型后自动过滤数据的时候就可以传入依赖项

```jsx
...

const data = [
  { category: "Fruits", price: "$1", stocked: true, name: "Apple" },
  { category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit" },
  { category: "Fruits", price: "$2", stocked: false, name: "Passionfruit" },
  { category: "Vegetables", price: "$2", stocked: true, name: "Spinach" },
  { category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin" },
  { category: "Vegetables", price: "$1", stocked: true, name: "Peas" }
]

function App () {

  const [category, setCategory] = useState('All')
  const [list, setList] = useState(data)

  useEffect(() => {
    setList(data.filter(item => (category === 'All' || item.category === category)))
  }, [category])

  return (
    <div>
      <select
        value={category}
        onChange={e => setCategory(e.target.value)}
      >
        <option value="All">All</option>
        <option value="Fruits">Fruits</option>
        <option value="Vegetables">Vegetables</option>
      </select>
      <ul>
        {
          list.map(item => (
            <li key={item.name}>
              <b>{ item.name }</b>
              <i 
                style={{ margin: '0 12px' }}
              >{ item.price }</i>
              <button
                disabled={!item.stocked}
              >Buy</button>
            </li>
          ))
        }
      </ul>
    </div>
  )
}

...
```

在这个案例中，我们给依赖项中放了一个`category`，然后这个`useEffect`就只会在初次渲染或`category`的值发生变化的时候会去执行，相当于类组件的`componentDidUpdate`，甚至还要更好用，因此我们也可以在`useEffect`中获取到数据更新后的值，最后再总结一下`useEffect`的执行顺序

```jsx
...

function App() {
  const [count, setCount] = useState(1)

  useEffect(() => {
    console.log('effect', count)

    return () => {
      console.log('clear effect', count)
    }
  }, [count])

  return (
    <div>
      <h1>{ count }</h1>
      <button
        onClick={() => setCount(x => x + 1)}
      >add</button>
    </div>
  )
}

/**
  组件初次渲染打印： effect 1

  点击按钮后打印
  clear effect 1
  effect 2
*/

...
```

即初次渲染或更新后会执行第一个参数函数，而其返回的函数会在下次更新前或卸载前执行。

### useRef

`useRef`返回一个`ref`对象，这个对象在组件的整个生命周期中持续存在。

语法：

```jsx
const refContainer = useRef(initialValue)
```

可以通过返回对象的`current`属性获取初始值(`initialValue`)。我们可以用它来获取dom节点

```jsx
...

function App () {

  const iptRef = useRef(null)

  const search = () => {
    // 通过 iptRef.current 获取dom节点 
    const searchText = iptRef.current.value;
    console.log(searchText)
  }

  return (
    <div>
      {/* 通过ref属性绑定dom */}
      <input
        ref={iptRef}
      />
      <button 
        onClick={search}
      >search</button>
    </div>
  )
}

...
```

上面案例中我们可以通过`iptRef.current`来获取到输入框的`dom`节点，从而获取输入框的内容。

`useRef`可以储存任意类型的变量，但是需要知道**直接修改`current`属性不会刷新页面**

```jsx
...

function Demo() {
  const countRef = useRef(1)

  const add = () => {
    countRef.current += 1
  }

  // 点击不会刷新视图
  return (
    <div onClick={add}>{ countRef.current }</div>
  )
}
```

但是可以通过一些手段让视图进行更新，比如下面这样

```jsx
...

function Demo() {
  const countRef = useRef(1)
  const [, set] = useState()

  const add = () => {
    set({}) // 使用set强制刷新视图
    countRef.current += 1
  }

  // 点击会刷新视图
  return (
    <div onClick={add}>{ countRef.current }</div>
  )
}
```

这样其实并没有什么太大的意义，如果需要视图随数据更新直接使用`useState`更加合适

### useReducer

`useReducer`这个hooks使用的并不算太多，但是一些场景用它确实要比`useState`更合适

语法：

```jsx
const [state, dispatch] = useReducer(reducer, initialState, [init])
```

其中，`reducer`必须是一个**纯函数**，而`initialState`则是我们仓库数据的初始状态，第三个参数`init`是可传的函数用于惰性初始化。而这个`hooks`返回一个数组，返回数组的第一项就是我们的仓库，第二项是一个`dispatch`，我们可以使用`dispatch`去派发一个`action`来更新仓库的数据。

在这里需要解释一下纯函数的定义：

1. 在函数输入相同的参数时，输出的结果相同。函数的输出和输入值以外的其他隐藏信息或状态无关，也和I/O设备产生的外部输出无关。
2. 该函数不能有语义化上可观察的函数副作用，诸如“触发事件”，使输出设备输出，或更改输出值以外物件的内容等。

可能看起来很复杂，但是其实就两点：

1. 确定的输入值一定会产生确定的输出。
2. 函数执行过程中不能产生副作用。

然后可以看一个阉割版的`todolist`案例

```jsx
...

// 定义初始化仓库
const initialState = {
  list: []
}

// 定义需要触发的action
const actions = {
  INSERT: "INSERT",
  DELETE: "DELETE",
  UPDATE: "UPDATE"
}

// 定义reducer
function reducer(state, action){
  const { type, payload } = action;
  switch(type){
    case actions.INSERT:
      return { 
        ...state,
        list: [...state.list, payload] 
      }
    case actions.UPDATE:
      return {  
        ...state,
        list: state.list.map(item => {
          return item.id === payload.id ? payload : item;
        })
      }
    case actions.DELETE:
      return {
        ...state,
        list: state.list.filter(item => item.id !== payload)
      }
    default:
      return state
  }
}

function Demo(){
  const [state, dispatch] = useReducer(reducer, initialState)

  const add = () => {
    dispatch({
      type: actions.INSERT,
      payload: { id: Date.now(), content: `INSERT at ${Date.now()}` }
    })
  }

  const del = (id) => {
    dispatch({
      type: actions.DELETE,
      payload: id
    })
  }

  const update = (id) => {
    dispatch({
      type: actions.UPDATE,
      payload: { id: id, content: `UPDATE at ${Date.now()}` }
    })
  }

  return (
    <div>
      <div>
        <button onClick={add}>ADD</button>
      </div>
      <ul>
        {
          state.list.map(item => {
            return (
              <li
                key={item.id}
              >
                { item.content }
                <button onClick={() => update(item.id)}>UPDATE</button>
                <button onClick={() => del(item.id)}>DELETE</button>
              </li>
            )
          })
        }
      </ul>
    </div>
  );
}
```

通过案例我们可以看到修改数据的操作只存在于`reducer`函数中，而在组件中我们只需要使用`dispatch`去派发不同的`action`就可以实现对数据不同的操作，并且我们可以发现案例中的`reducer`中在处理对应的`action`时都返回了一个新的对象，这是因为`useReducer`是对数据的内存地址进行的比较，如果内存地址没有发生改变的话视图是不会更新的，比如:

```jsx
...

function reducer(state, action){
  const { type, payload } = action;
  switch(type){
    case actions.INSERT:
      state.list = [...state.list, payload]
      return state // 返回的还是原对象，useReducer会以为数据没有变化，不会更新视图
    // 省略...
  }
}

...
```

在源码上`useReducer`的实现和`useState`是一样的，只是`useState`有一个默认的`action`而`useReducer`则是需要我们自己定义触发的`action`以及其对应的逻辑，在工作中笔者发现大多同事，包括我自己在内使用这个`hooks`都使用的并没有那么频繁，不过在处理复杂数据的时候`useReducer`确实要比`useState`好一些。

另外就是惰性初始化，比如上面的例子如果用惰性初始化就需要改成这个样子

```jsx
...

const init = (list) => ({ list })

function Demo(){
  const [state, dispatch] = useReducer(reducer, [], init)

  ...
}
```

第三个参数`init`会把第二个参数作为入参去计算出初始的`state`，这样便于后期重置`state`，最后`useReducer`是一个高级`hooks`没有它我们依旧可以完成需求，但是使用它可以增加代码的可读性。

## 自定义hooks

随着自己使用react的时间越来越长，以及业务复杂度的提高，自己也是见识到了越来越多的优秀的react代码，也对hooks有了更深一步对理解。

将业务逻辑抽离成为自定义hooks之后我们的组件内部会变得非常的干净，在后续维护的时候会轻松很多。

举个🌰：

我们有一个列表页，假设这个页面里只有一个列表，那么常规写法如下：

```jsx
import React, { useState, useEffect } from 'react'
import { Table, Button, Space, Tag } from 'antd'

const columns = [
  {
    title: 'Name',
    dataIndex: 'name',
    key: 'name',
    render: text => <a>{text}</a>,
  },
  {
    title: 'Age',
    dataIndex: 'age',
    key: 'age',
  },
  {
    title: 'Address',
    dataIndex: 'address',
    key: 'address',
  },
  {
    title: 'Tags',
    key: 'tags',
    dataIndex: 'tags',
    render: tags => (
      <>
        {tags.map(tag => {
          let color = tag.length > 5 ? 'geekblue' : 'green';
          if (tag === 'loser') {
            color = 'volcano';
          }
          return (
            <Tag color={color} key={tag}>
              {tag.toUpperCase()}
            </Tag>
          );
        })}
      </>
    ),
  },
  {
    title: 'Action',
    key: 'action',
    render: (text, record) => (
      <Space size="middle">
        <a>Invite {record.name}</a>
        <a>Delete</a>
      </Space>
    ),
  },
];

// 假数据
const data = [
  {
    key: '1',
    name: 'John Brown',
    age: 32,
    address: 'New York No. 1 Lake Park',
    tags: ['nice', 'developer'],
  },
  {
    key: '2',
    name: 'Jim Green',
    age: 42,
    address: 'London No. 1 Lake Park',
    tags: ['loser'],
  },
  {
    key: '3',
    name: 'Joe Black',
    age: 32,
    address: 'Sidney No. 1 Lake Park',
    tags: ['cool', 'teacher'],
  },
];

// 使用Promise和setTimeout模拟请求
const getData = () => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(data)
    }, 2000)
  })
}

// 组件
const Demo = () => {
  const [loading, setLoading] = useState(false)
  const [data, setData] = useState([])
 
 useEffect(() => {
   initTable()
 }, [])

  const initTable = async () => {
    setLoading(true)
    const res = await getData();
    setData(res)
    setLoading(false)
  }

  return (
    <div>
      <Button
        onClick={() => initTable()}
      >刷新</Button>

      <Table
        columns={columns}
        dataSource={data}
        loading={loading}
      />
    </div>
  )
}

export default Demo;
```

笔者以前也是经常使用这种写法，但是随着页面中功能当日益丰富后面就发现这种写法会导致组件看起来过于臃肿，业务复杂的时候维护起来十分的麻烦。

然后笔者喜欢在没有需求的时候查看其他同事的代码，来学习其他人代码中比较好的设计思路，然后发现了一些大佬们的思路，然后醍醐灌顶，发现hooks原来应该这样用。

```jsx
// 自定义hooks
function useData () {
  const [loading, setLoading] = useState(false)
  const [data, setData] = useState([])
  const [error, setError] = useState(null)

  useEffect(() => {
    run()
  }, [])

  const run = async (params = {}) => {
    try {
      setLoading(true)
      const res = await getData(params);
      setData(res)
      setLoading(false)
    } catch (error) {
      setLoading(false)
      setError(error)
    }
  }

  return {
    loading,
    data,
    error,
    run
  }
}
```

经过这样的抽离后，使用起来就会十分的方便。

```js
...

function Demo () {

  const { loading, data, run } = useData()

  return (
    <div>
      <Button
        onClick={() => run()}
      >刷新</Button>

      <Table
        columns={columns}
        dataSource={data}
        loading={loading}
      />
    </div>
  )
}

...
```

可以看到在使用的地方我们只需要从我们封装的`hooks`里将数据结构出来即可，这样组件内部也变得十分的清爽，抽离出来的自定义`hooks`要以`use`开头，遵循`hooks`规范。

并且如果其他组件如果也需要调用这个接口的话直接使用我们封装的这个`hooks`就可以，真的是极大的提高了代码的复用性。

## 常见的自定义hooks

经过上面的🌰，我们就会发现封装的好处，然后笔者在这里记录了一些常用的自定义`hooks`，有些地方设计可能有些不妥，欢迎大家指出。

### 防抖hooks useDebounceFn

防抖和节流是日常开发中非常常见的功能，而我们有时候希望我们的写的方法拥有防抖的效果，这时候就可以把防抖的逻辑抽离成一个`hooks`，以便于以后的复用。

```jsx
import { useRef, useCallback } from 'react'

/**
 * 防抖
 * @param {Function} fn 
 * @param {Number} ms 
 * @param {Array} deps 
 * @returns Function
 */
function useDebounceFn(fn, ms, deps = []) {
  let timer = useRef()
  
  const debounce = useCallback((arg) => {
    timer.current && clearTimeout(timer.current)

    timer.current = setTimeout(() => {
      fn(arg)
    }, ms)
  }, [fn, ms, ...deps])

  return debounce
}
```

使用

```jsx
...

function App() {

  const [text, setText] = useState("")

  const handleInput = useDebounceFn((v) => {
    setText(v)
  }, 500)

  return (
    <div className="App">
      <input
        onInput={e => handleInput(e.target.value)}
      />
      { text }
    </div>
  )
}

...
```

### 节流hooks useThrottleFn

有防抖必然有节流，节流同样可以抽离成为一个单独的`hooks`，先来实现一个基本的节流`hooks`。

```jsx
...

function useThrottleFn(fn, ms, deps = []) {
  const timer = useRef(0)

  const throttle = useCallback((arg) => {
    let now = Date.now()
    if(now - timer.current > ms){
      fn(arg)
      timer.current = now
    }
  }, [timer.current, ...deps])

  return throttle
}

...
```

这样看起来好像是没什么问题，但是在我们平时的使用场景中，有些时候是希望节流方法在不能使用的时候给予一个友好的提示的，所以需要再扩展一下。

```jsx
...

/**
 * 节流
 * @param {Function} fn 
 * @param {Object|Number} options 
 * @param {Array} deps 
 * @returns Function
 */
function useThrottleFn(fn, options = {}, deps = []) {
  const timer = useRef(0)
  const ms = typeof options === 'number' ? options : options.ms

  const throttle = useCallback((arg) => {
    let now = Date.now()
    if(now - timer.current > ms){
      fn(arg)
      timer.current = now
    } else {
      options?.onError && options.onError()
    }
  }, [timer.current, ...deps])

  return throttle
}

...
```

使用

```jsx
...
import { Table, Button, message } from 'antd'

function App() {

  const columns = [
    {
      title: "ID",
      dataIndex: "id"
    }
  ]

  const [data, setData] = useState([])

  const search = useThrottleFn(() => {
    setData([{ id: Math.random() }])
  }, {
    ms: 2000,
    onError: () => {
      message.error('操作太频繁了，请稍后再试！')
    }
  })

  return (
    <div>
      <Button
        onClick={() => search()}
      >Search</Button>

      <Table
        columns={columns}
        dataSource={data}
      />
    </div>
  )
}

...
```