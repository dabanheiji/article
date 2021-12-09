>黑鸡虽然一直在使用`react`框架，但是黑鸡一直使用的都是js模板，从来没有使用过ts来写项目，所以准备使用`react+ts`来实现一个`TodoList`来学习一下使用ts写`react`

## 搭建项目

黑鸡比较喜欢尤大的`vite`所以黑鸡就使用了`vite`搭建项目，当然使用`webpack`也是可以的

```
yarn create @vitejs/app
```

选择ts模板然后删除`App.tsx`中内容，我们先设计一下组件目录，todolist分为上部一个输入框和下面一个列表渲染，故而拆分成一个`TdInput`组件和一个`List`组件，故组件目录如下

```
|- src
    |- components
        |- TodoList
            |- index.tsx //主组件
            |- TdInput.tsx //TdInput组件
            |- List.tsx // 列表组件
            |- Item.tsx // 列表项组件
```

## 嵌套组件

ok，准备工作都执行完毕了，接下来写`App.tsx`中的代码

```jsx
// App.tsx
import React, {FC, ReactElement} from 'react'

import TodoList from './components/TodoList'

const App:FC = ():ReactElement => {
    return (
        <div>
            <TodoList/>
        </div>
    )
}

export default App
```

接下来开始写`TodoList`中的代码，TodoList是上面一个输入框，下面一个列表的展示格式，所以这个组件布局比较简单，先将组件嵌套完毕，然后再逐一实现功能

```jsx
// components/TodoList/index.tsx
import React, {FC, ReactElement} from 'react'

import TdInput from './TdInput'
import List from './List'

const TodoList: FC = (): ReactElement => {
    return (
        <div>
        	<TdInput/>
            <List/>
        </div>
    )
}

export default TodoList
```

这样组件之间的嵌套关系就处理完毕，然后开始逻辑和细节的实现

## 实现TdInput组件

我们先来实现一下TdInput组件

```jsx
// components/TodoList/TdInput.tsx
import React, {FC, ReactElement} from 'react'

const TdInput:FC = ():ReactElement => {
    return (
    	<div>
        	<input type="text" placeholder="请输入代办项"></input>
            <button>添加</button>
        </div>
    )
}

export default TdInput
```

这样就写好了基本结构，然后我们分析一下这个组件的功能，点击添加按钮应该是要添加一个代办事项的，并且添加代办项这个方法得是从`props`中传入的，并且我们还需要判断这个事项是否是存在的，如果存在则不让添加，所以我们还需要使用全部的数据，分析之后我们需要规范一下每一条代办项的数据类型，所以我们在TodoList文件夹下创建一个`typings`文件夹来定义一些用到的接口

```
|- src
    |- components
        |- TodoList
        	|- typings  +
        		|- index.ts  +
            |- index.tsx //主组件
            |- TdInput.tsx //TdInput组件
            |- List.tsx // 列表组件
            |- Item.tsx // 列表项组件
```

```js
// components/TodoList/typings/index.ts
export interface ITodo {
    id: number;
    content: string;
    done: boolean;
}
```

定义好这个接口之后我们还需要在组件中定义一个`props`的规范接口所以`TdInput`组件中的代码变成

```jsx
// components/TodoList/TdInput.tsx
import React, { FC, ReactElement, useRef } from 'react'
import { ITodo } from './typings'

interface IProps {
    addTodo: ( todo: ITodo )=>void; // 添加代办项的方法从父组件中传入
    todoList: ITodo[]; // 判断代办项是否已存在需要全部数据
}

const TInput: FC<IProps> = ({
    addTodo,
    todoList
}): ReactElement => {

    const inputRef = useRef<HTMLInputElement>(null)

    const addItem = ():void => {
        const val: string = inputRef.current!.value.trim()
    
        if(val.length){
            const isExist = todoList.find(todo => todo.content === val)

            if(isExist){
                alert('已存在')
                return 
            }

            addTodo({
                id: new Date().getTime(),
                content: val,
                done: false
            })

            inputRef.current!.value = ''
        }
    }

    return (
        <div className="t-input">
            <input type="text" ref={inputRef} />
            <button onClick={ addItem }>添加</button>
        </div>
    )
}

export default TInput
```

这样`TdInput`组件就完成了

## 完成list组件

先分析组件的功能，`List`组件只是一个渲染，但是在`Item`组件中需要使用到删除和修改状态的方法，而这些方法都应该从`TodoList`中传入，故`List`组件代码如下

```jsx
// components/TodoList/List.tsx
import React, { FC, ReactElement } from 'react'

import Item from './Item'
import { ITodo } from './typings'

interface Iprops {
    todoList: ITodo[]; // 需要渲染的数据
    removeItem: (id:number)=>void; // 删除单独一项的方法
    handlerItem: (id:number)=>void; // 修改某项状态的方法
}

const List: FC<Iprops> = ({
    todoList,
    removeItem,
    handlerItem
}): ReactElement => {
    return (
        <div>
            {
                todoList.map(todo => {
                    return (
                        <Item 
                            todo={todo}
                            key={todo.id}
                            removeItem={removeItem}
                            handlerItem={handlerItem}
                        />
                    )
                })
            }
        </div>
    )
}

export default List
```

然后写`Item`组件，这里就也很简单了，直接调用传入的方法就可以了

```jsx
// components/TodoList/Item.tsx
import React, { FC, ReactElement } from 'react'
import { ITodo } from './typings'

interface Iprops {
    todo: ITodo;
    removeItem: (id: number) => void;
    handlerItem: (id: number) => void; 
}

const Item: FC<Iprops> = ({
    todo,
    removeItem,
    handlerItem
}): ReactElement => {

    const { id, content, done } = todo

    return (
        <div>
            <input 
                type="checkbox" 
                checked={done} 
                onChange={()=>handlerItem(id)}
            />
            <span
                style={{ textDecoration: done ? 'line-through' : 'none' }}
            >{ content }</span>
            <button onClick={()=>removeItem(id)}>删除</button>
        </div>
    )
}

export default Item
```

到这里组件就全部写完了，接下来是业务逻辑的编写

## 编写业务逻辑

写完了组件接下来回到`TodoList`组件，在这里我们需要实现所有的方法并传递给两个子组件，首先我们需要一个有状态的数据，可以使用`useState`但是黑鸡在这里就不使用`useState`了，黑鸡在这里选择了`useReducer`，`useReducer`需要一个初始仓库故我们先声明一个仓库，并定义一下仓库的接口规范，除了仓库之外还需要一个`reducer`函数，故我们创建一个`reducer`文件夹在下面写上这个函数

```js
// components/TodoList/typings/index.ts
...

export interface IState {
    todoList: ITodo[]
}

export interface IAction {
    type: ACTION_TYPE,
    payload: ITodo | number
}

export enum ACTION_TYPE {
    ADD_TODO = 'addTodo',
    REMOVE_TODO = 'removeTodo',
    HANDLER_TODO = 'handlerTodo'
}
```

```js
// components/TodoList/reducer/index.ts
import { ACTION_TYPE, IAction, IState, ITodo } from "./typings";

function todoReducer(state: IState, action: IAction): IState{
    const { type, payload } = action

    switch(type){
        case ACTION_TYPE.ADD_TODO: //添加代办项
            return {
                ...state,
                todoList: [...state.todoList, payload as ITodo]
            }
        case ACTION_TYPE.REMOVE_TODO: //删除代办项
            return {
                ...state,
                todoList: state.todoList.filter(todo => todo.id !== payload)
            }
        case ACTION_TYPE.HANDLER_TODO: //修改代办项状态
            return {
                ...state,
                todoList: state.todoList.map(todo => {
                    return todo.id === payload ?
                    {
                        ...todo,
                        done: !todo.done
                    } :
                    {
                        ...todo
                    }
                })
            }
        default: 
            return {
                ...state
            }
    }
}

export {
    todoReducer
}
```

ok，业务逻辑写完之后我们只需要在`TodoList`中调用`dispatch`传不同的参数来实现不同的业务

```jsx
// components/TodoList/index.tsx
import React, { ReactElement, FC, useCallback, useReducer } from 'react'

import TInput from './TInput'
import List from './List'

import { todoReducer } from './reducer'
import { ITodo, IState, ACTION_TYPE } from './typings'

const initalState: IState = {
    todoList: []
}

const TodoList: FC = (): ReactElement => {

    const [state, dispatch] = useReducer(todoReducer, initalState)

    const addTodo = useCallback((todo: ITodo) => {
        dispatch({
            type: ACTION_TYPE.ADD_TODO,
            payload: todo
        })
    }, [])

    const removeItem = useCallback((id: number)=>{
        dispatch({
            type: ACTION_TYPE.REMOVE_TODO,
            payload: id
        })
    }, [])

    const handlerItem = useCallback((id: number)=>{
        dispatch({
            type: ACTION_TYPE.HANDLER_TODO,
            payload: id
        })
    }, [])

    return (
        <div className="todolist">
            <TInput
                addTodo={addTodo}
                todoList={state.todoList}
            />
            <List
                todoList={state.todoList}
                removeItem={removeItem}
                handlerItem={handlerItem}
            />
        </div>
    )
}

export default TodoList
```

黑鸡在学习任何东西的时候都喜欢先写一个`TodoList`，通过这次的写代码黑鸡也是对ts有了一些更好的掌握，以后继续努力，up！up！up！