# hooks

## 自定义hooks

随着自己使用react的时间越来越长，自己也是见识到了越来越多的优秀的react代码，也对hooks有了更深一步对理解。

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

可以看到在使用的地方我们只需要从我们封装的`hooks`里将数据结构出来即可，这样组件内部也变得十分的清爽。

并且如果其他组件如果也需要调用这个接口的话直接使用我们封装的这个`hooks`就可以，真的是极大的提高了代码的复用性。

## 常见的自定义hooks

### 防抖hooks useDebounceFn

防抖和节流是日常开发中非常常见的功能，所以非常值得封装。

```jsx
import { useRef } from 'react'

function useDebounceFn(fn, ms) {
  let timer = useRef()
  
  const debounce = (e) => {
    timer.current && clearTimeout(timer.current)

    timer.current = setTimeout(() => {
      fn(e)
    }, ms)
  }

  return debounce
}
```
