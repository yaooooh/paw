---
title: React 学习笔记
date: 2025-04-19 20:31:42
tags: [React]
categories: [编程]
---

# React 学习



## 第一站

一个简单的例子

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <div id="app"></div>
  
  <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
  <!-- 添加对 jsx 语法的支持 -->
  <script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>

  <script type="text/babel">
    const app = ReactDOM.createRoot(document.querySelector('#app'))
    let text = 'Hello World'

    function changeText() {
      text = 'Hello React'
      render()
    }

    function render() {
      app.render(
        <div>
          <h1>{text}</h1>
          <button onClick={changeText}>Click</button>
        </div>
      )
    }

    render()
  </script>
</body>
</html>
```



### 使用类的方式创建组件

```jsx
import React, { Component } from "react"
import About from './About'

export default class extends Component {
  constructor() {
    super()
    this.state = {
      message: 'Hello world!',
      count: 0
    }
  }

  componentDidMount() {
    console.log('after component mount')
  }

  componentDidUpdate() {
    console.log('after component update')
  }

  componentWillUnmount() {
    console.log('before component unmount')
  }

  render() {
    const {count} = this.state

    return (
      <div>
        <h1>{this.state.message}</h1>
        <About></About>
        <h2>{count}</h2>
        <button onClick={() => this.setState({count: count + 1})}>Add One</button>
      </div>
    )
  }
}
```

#### 类组件的声明周期

- construct：在组件被实例化的时候执行
  - 如果不初始化 state 或不进行方法绑定，则不需要为 React 组件实现构造函数
  - 通过给 this.state 赋值对象来初始化内部的 state
  - 为事件绑定实例（this）
- render：组件实例化或更新界面中的变量都会被重新执行
- componentDidMount：组件被挂载后会执行
  - 依赖于 DOM 的操作可以在这里进行
  - 在此处发送网络请求就是最好的地方
  - 可以在此处添加一些订阅（会在 componentWillUnmount 取消订阅）
- componentDidUpdate：组件被更新后会执行（render 先于此函数执行）
  - 当组件更新后，可以在此处对 DOM 进行操作
  - 如果对更新前后的 props 进行了比较，也可以选择在此处进行网络请求；（例如，当 props 未发生变化时，则不会执行网络请求）
- componentWillUnmount：组件被销毁前执行
  - 在此方法中执行必要的清理操作，例如：清除 timer，取消网络请求或清除在 componentDidMount 中创建的订阅等

#### 父子组件通信

Father component

```jsx
import React, { Component } from 'react'
import Child from './Child'

export default class Father extends Component {
  constructor() {
    super()
    this.state = { count: 0 }
  }

  render() {
    const { count } = this.state
    return (
      <div>
        <h2>Father</h2>
        <p>{count}</p>

        <Child data={['father transfer data1', 'father transfer data2']}
               title={123}
               changeNumber={(num) => this.setState({count: count + num})} />
      </div>
    )
  }
}
```

Child component

```jsx
import { Component } from 'react'
import PropTypes from 'prop-types'

export class Child extends Component {
  // 可以通过这种方式设置默认值，此设置将会覆盖外部设置的所有默认值
  static defaultProps = {
    innerDefaultValue: 'inner default value'
  }

  constructor(props) {
    super(props)
    this.state = {}
  }

  addNumber(num) {
    this.props.changeNumber(num)
  }

  render() {
    const {data, title, defaultValue, innerDefaultValue} = this.props

    return (
      <div>
        <h2>Child</h2>
        <p>{title}</p>
        <p>{defaultValue}</p>
        <p>{innerDefaultValue}</p>
        <button onClick={() => this.addNumber(1)}>Add 1</button>
        <button onClick={() => this.addNumber(-1)}>Dec 1</button>
        <ul>
          {
            data?.map(item => <li key={item}>{item}</li>)
          }
        </ul>
      </div>
    )
  }
}

// 未传入属性进行限制
Child.propTypes = {
  data: PropTypes.array.isRequired,
  title: PropTypes.string,
  changeNumber: PropTypes.func.isRequired
}

// 设置默认值
// Child.defaultProps = {
//   defaultValue: 'default'
// }

export default Child
```



### 使用函数式定义组件

```jsx
/*
  1. 函数组件没有声明周期
  2. this 关键字不能指向组件实例（因为没有组件实例）
  3. 没有内部状态
*/
export default function About() {
  return <div>About</div>
}
```



### 对参数进行校验

对于传递给子组件的数据，有时候我们可能希望进行验证

- 项目中默认继承了 Flow 或者 TypeScript，那么直接就可以进行类型验证
- 但是，即使我们没有使用 Flow 或者 TypeScript，也可以通过 prop-types 库来进行参数验证

安装 prop-types 库

```bash
npm install prop-types -D
```

在组件传值进行类型限制

```jsx
import React, { Component } from 'react'
import PropTypes from 'prop-types'

export class Child extends Component {
  constructor(props) {
    super(props)
  }

  render() {
    const {data, title, defaultValue} = this.props

    return (
      <div>
        <h2>Child</h2>
        <p>{title}</p>
        <p>{defaultValue}</p>
        <ul>
          {
            data?.map(item => <li key={item}>{item}</li>)
          }
        </ul>
      </div>
    )
  }
}

// 未传入属性进行限制
Child.propTypes = {
  data: PropTypes.array.isRequired,
  title: PropTypes.string
}

// 设置默认值
Child.defaultProps = {
  defaultValue: 'default'
}

export default Child
```



### 插槽

在插槽组件中拿到 this.props 的 children 属性，该属性包含了父组件在使用该组件传递的元素

插槽组件

```jsx
import { Component } from 'react'

import './index.css'

export default class extends Component {
  render() {
    // 接收参数一
    // const {children} = this.props

    // return (
    //   <div className='nav-bar'>
    //     <div className='left'>{children[0]}</div>
    //     <div className='center'>{children[1]}</div>
    //     <div className='right'>{children[2]}</div>
    //   </div>
    // )

    // 接收参数二
    const {leftSlot, centerSlot, rightSlot} = this.props
    return (
      <div className='nav-bar'>
        <div className="left">{leftSlot('left')}</div>
        <div className="center">{centerSlot}</div>
        <div className="right">{rightSlot}</div>
      </div>
    )
  }
}
```

父组件

```jsx
import { Component } from "react"

import NavBar from "../NavBar"

export default class extends Component {
  render() {
    // 传递方式一
    // return (
    //   <div>
    //     <h1>Hello world</h1>
    //     <NavBar>
    //       {/* 当只传递一个元素时，子元素接收到 children 不为数组，只为该元素 */}
    //       <button>left button</button>
    //       <input type="text" />
    //       <i>三</i>
    //     </NavBar>
    //   </div>
    // )

    // 传递方式二
    return (
      <div>
        <h1>Hello world</h1>
        {/* 通过使用函数的方式达到作用域插槽的效果 */}
        <NavBar leftSlot={text => <button>{text}</button>}
                centerSlot={<input />}
                rightSlot={<i>三</i>}/>
      </div> 
    )
  }
}
```



### 跨组件通信

1. 定义上下文对象

   主题上下文对象

```js
import { createContext } from "react"

// 设置默认值，当需要使用该上下文的组件并没有被 Provider 包裹时，将会显示该值
const ThemeContext = createContext({color: '#999', backgroundColor: '#eee'})

export default ThemeContext
```

​	用户上下文

```jsx
import { createContext } from "react"

const UserContext = createContext()

export default UserContext
```

1. 在需要传递数据的组件中使用该上下文并传递数据

   ```jsx
   import { Component } from "react"
   
   import ThemeContext from './context/ThemeContext'
   import UserContext from './context/UserContext'
   import Father from "./components/Father"
   
   
   export default class extends Component {
     render() {
       return (
         <div>
           <h1>Hello world</h1>
           {/* 将需要使用到该上下文的元素使用 Provider 包裹，并传入 value */}
           <UserContext.Provider value={{username: 'zhangsan', age: 18}}>
             <ThemeContext.Provider value={{color: 'red', backgroundColor: '#9900aa'}}>
               <Father />
             </ThemeContext.Provider>
           </UserContext.Provider>
         </div>
       )
     }
   }
   ```

2. 在需要引用该值的组件中导入该上下文对象，并设置该子组件的 contextTypes 属性为该上下文对象

   ```jsx
   import React, { Component } from 'react'
   
   import ThemeContext from '../../context/ThemeContext'
   import UserContext from '../../context/UserContext'
   
   class Child extends Component {
     render() {
       // 获取该上下文对象的所有值
       const {color, backgroundColor} = this.context
   
       return (
         <div style={{color: color, backgroundColor: backgroundColor}}>
           <h3>Child</h3>
           <UserContext.Consumer>
             {
               user => {
                 return (
                   <ul>
                     <li>username: {user.username}</li>
                     <li>age: {user.age}</li>
                   </ul>
                 )
               }
             }
           </UserContext.Consumer>
         </div>
       )
     }
   }
   // 设置该组件的 contextType 为上下文对象
   Child.contextType = ThemeContext
   
   export default Child
   ```

   在函数式组件中使用该上下文

   ```jsx
   import ThemeContext from "../../context/ThemeContext"
   
   function ChildMain () {
     return(
       <div>
         <ThemeContext.Consumer>
           {
             value => {
               return <div style={{color: value.color, backgroundColor: value.backgroundColor}}>
                 child main
               </div>
             }
           }
         </ThemeContext.Consumer>
       </div>
     )
   }
   
   export default ChildMain
   ```

   

### 事件总线

事件总线类的定义

```js
class EventBus {

  eventList = new Map()

  /**
   * 触发事件
   * @param {String} eventName 
   * @param  {...any} args 
   */
  emit(eventName, ...args) {
    if (!this.eventList.has(eventName)) {
      throw new Error('No such event emit')
    }
    const funs = this.eventList.get(eventName)
    for (let fun of funs) {
      fun(...args)
    }
  }

  /**
   * 监听事件
   * @param {String} eventName 
   * @param {Function} callback 
   * @param {Object} that 
   */
  on(eventName, callback, that) {
    if (this.eventList.has(eventName)) {
      this.eventList.get(eventName).add(callback.bind(that))
    } else {
      this.eventList.set(eventName, new Set([callback.bind(that)]))
    }
  }

  /**
   * 销毁事件
   * @param {String} eventName 
   */
  off(eventName) {
    if (!this.eventList.has(eventName)) {
      throw new Error('No such event emit')
    }
    this.eventList.delete(eventName)
  }
}

export default EventBus
```

定义一个事件总线类

```js
import EventBus from "@/utils/eventBus"

export default new EventBus()
```

发送事件组件使用 emit 发送事件

```jsx
import { Component } from "react"

import eventBus from '@/events/EventBus'

export default class extends Component {

  constructor() {
    super()
    this.state = {
      title: 'Hello Father'
    }
  }

  componentDidMount() {
    console.log('Father component Mounted')
  }

  componentDidUpdate() {
    console.log('Father component Updated')
  }

  changeTitle() {
    eventBus.emit('UpdateTitle', {name: 'zhangsan', age: 19})
    this.setState({
      title: 'Hello Father Component'
    })
  }

  render() {
    const { title } = this.state

    return (
      <div>
        <h2>Father Component</h2>
        <button onClick={() => this.changeTitle()}>{title}</button>
      </div>
    )
  }
}
```

监听事件组件使用 on 监听事件

```jsx
import React, { Component } from 'react'

import Father from '@components/Father'
import eventBus from '@/events/EventBus'

export default class App extends Component {

  constructor() {
    super()
    this.state = {
      appName: 'App'
    }
  }

  componentDidMount() {
    console.log('App component Mounted')
    eventBus.on('UpdateTitle', (args) => {
      console.log(args)
      this.setState({
        appName: 'App Component'
      })
    })
  }

  componentDidUpdate() {
    console.log('App component updated')
  }

  render() {
    const { appName } = this.state

    return (
      <div>
        <h1>{appName}</h1>
        <Father></Father>
      </div>
    )
  }
}
```

输出结果

```bash
Father component Mounted
App component Mounted
{name: 'zhangsan', age: 19}
Father component Updated
App component updated
```

// TODO 由此可以推断出，挂载组件从内往外，更新组件从外往内



### setState 函数

setState 为异步函数，设计为异步函数可以显著提升性能

- 如果每次调用 setState 都进行一次更新，那么意味着 render 函数会被频繁调用，界面重新渲染这样效率很低
- 最好的办法应该是获取到多个更新，之后进行批量更新
- 如果同步更新了 state，但是还没有执行 render 函数，那么 state 和 props 不能保持同步
  - state 和 props 不能保持一致性，在开发过程中产生很多的问题

```jsx
import { Component } from "react"

export default class extends Component {

  constructor() {
    super()
    this.state = {
      count: 10
    }
  }

  componentDidUpdate() {
    console.log('Component updated')
  }

  addCount() {
    // 通过回调的方式处理更新的数据
    this.setState((state, props) => {
      // 可以编写一些对新的 state 的处理逻辑，
      // 可以获得之前的 state 和 props 对象
      console.log(state, props)
      return {
        count: state.count + 1
      }
    })

    // setState 是一个异步函数，在这个地方将会打印修改之前的 state
    console.log(this.state)

    // 通过传入回调函数，该回调函数将会在更新之后执行
    this.setState({
      count: this.state.count + 1
    }, () => {
      // componentDidUpdate 声明周期函数将会先于这个函数执行
      console.log(this.state)
    })

    // 在 react 18 之前如下代码（setTimeout 或者 原生 dom 事件中 或者 promise 中）将会输出修改后的，也即 setState 变成同步函数
    // 在 react 18 及之后，如下代码仍然是异步，输出也是修改前的 count
    setTimeout(() => {
      this.setState({count: this.state.count + 1})
      console.log('setTimeout function', this.state.count)
    }, 0)

    document.querySelector('button').onclick = () => {
      this.setState({count: this.state.count + 1})
      console.log('onclick event', this.state.count)
    }
  }

  render() {
    const {count} = this.state
    const {title} = this.props

    return (
      <div>
        <h1>{title}</h1>
        <p>{count}</p>
        <button onClick={() => this.addCount()}>+</button>
      </div>
    )
  }
}
```

将 setState 变为同步可以通过 flushSync 函数

```jsx
import { Component } from "react"
import { flushSync } from "react-dom"

export default class extends Component {

  constructor() {
    super()
    this.state = {
      count: 10
    }
  }

  componentDidUpdate() {
    console.log('Component updated')
  }

  addCount() { 
    flushSync(() => {
      this.setState({
        count: this.state.count + 1
      })
    })
    // 这里将会拿到更新之后的结果
    console.log(this.state.count)
  }

  render() {
    const {count} = this.state
    const {title} = this.props

    return (
      <div>
        <h1>{title}</h1>
        <p>{count}</p>
        <button onClick={() => this.addCount()}>+</button>
      </div>
    )
  }
}
```



### SCU（should component update）

```jsx
import { Component } from "react"
import { flushSync } from "react-dom"

export default class extends Component {

  constructor() {
    super()
    this.state = {
      count: 10
    }
  }

  componentDidUpdate() {
    console.log('Component updated')
  }

  // 此种方式成为 SCU should component update
  // 当此函数返回 false 时不会重新渲染，当返回 true 时会重新渲染
  shouldComponentUpdate(nextProps, nextState) {
    console.log(nextProps, nextState, nextProps == this.props, nextState.count == this.state.count)
    if (nextProps == this.props && nextState.count == this.state.count) {
      // 不进行重新渲染
      return false
    }
    return true
  }

  addCount() { 
    flushSync(() => {
      this.setState({
        count: this.state.count + 1
      })
    })
    // 这里将会拿到更新之后的结果
    console.log(this.state.count)
  }

  render() {
    const {count} = this.state
    const {title} = this.props

    return (
      <div>
        <h1>{title}</h1>
        <p>{count}</p>
        <button onClick={() => this.addCount()}>+</button>
      </div>
    )
  }
}
```

### PureComponent 和 memo

> PureComponent 相当于使用 shouldComponentUpdate，其在该函数中进行比较 state 和 props.
>
> 其比较的为浅层的，当比较的是引用类型的数据时，其内部属性发生改变，将不会调用 render 函数
>
> 因此在对 state 中的属性进行修改时，应创建新对象
>
> memo 为 函数式组件的 PureComponent

```jsx
import { PureComponent } from "react"
import Child from "@components/Child"
import ChildFunction from "@components/ChildFunction"

export default class extends PureComponent {

  constructor() {
    super()
    this.state = {
      count: 10
    }
  }

  componentDidUpdate() {
    console.log('app updated')
  }

  addCount() { 
    this.setState({
      count: this.state.count + 1
    })
  }

  render() {
    const {count} = this.state
    const {title} = this.props
    console.log('App render')

    return (
      <div>
        <h1>{title}</h1>
        <p>{count}</p>
        <button onClick={() => this.addCount()}>+</button>
        <Child count={count}></Child>
        <ChildFunction message={title}></ChildFunction>
      </div>
    )
  }
}
```

Child

```jsx
import { PureComponent } from "react"

export default class extends PureComponent {

  constructor() {
    super()
    this.state = {
      message: 'child message'
    }
  }

  componentDidUpdate() {
    console.log('child update')
  }

  render() {
    console.log('child render')
    return (
      <div>
        <h2>Child</h2>
        <p>{this.state.message}</p>
      </div>
    )
  }
}
```

ChildFunction

```jsx
import { memo } from "react"

const ChildFunction = memo(function(props) {
  return (
    <div>
      <h2>Child Function</h2>
      <p>{props.message}</p>
    </div>
  )
})

export default ChildFunction
```

修改对象触发 render 更新

```jsx
import { PureComponent } from "react"
import Child from "@components/Child"
import ChildFunction from "@components/ChildFunction"

export default class extends PureComponent {

  constructor() {
    super()
    this.state = {
      count: 10,
      books: [
        {name: 'book1', price: 80, count: 1},
        {name: 'book2', price: 78, count: 1},
        {name: 'book3', price: 90, count: 3},
      ]
    }
  }

  componentDidUpdate() {
    console.log('app updated')
  }

  addCount() { 
    this.setState({
      count: this.state.count + 1
    })
  }

  addBook() {
    const books = [...this.state.books]
    books.push({name: 'book4', price: 77, count: 1})
    this.setState({
      books: books
    })
  }

  addBookCount(index) {
    const books = [...this.state.books]
    books[index].count ++
    this.setState({
      books: books
    })
  }

  render() {
    const {count, books} = this.state
    const {title} = this.props
    console.log('App render')

    return (
      <div>
        <h1>{title}</h1>
        <p>{count}</p>
        <button onClick={() => this.addCount()}>+</button>
        <Child count={count}></Child>
        <ChildFunction message={title}></ChildFunction>
        <hr />
        <ul>
          {
            books.map((book, index) => {
              return (
                <li key={index}>
                  {book.name}-{book.price}-{book.count}
                  <button onClick={() => this.addBookCount(index)}>Add 1</button>
                </li>
              )
            })
          }
        </ul>
        <button onClick={() => this.addBook()}>Add New Book</button>
      </div>
    )
  }
}
```



### 获取组件实例 ref

父组件

```jsx
import React, { createRef, PureComponent } from 'react'
import Child from '@components/Child'
import ChildFunction from './components/ChildFunction'

export class App extends PureComponent {

  constructor() {
    super()
    
    // 推荐使用这种方式
    this.node = createRef(),
    this.node2 = null
    this.child = createRef()
    this.functionChild = createRef()
  }

  componentDidMount() {
    console.log(this.node.current)
    console.log(this.node2)
    this.child.current.test()
    console.log(this.child.current, this.child.current.test)
    console.log(this.functionChild.current)
  }

  btnClick(e) {
    console.log(e.target)
    console.log(this.refs)
  }

  render() {

    return (
      <div>
        <h1>App</h1>
        <div ref={this.node}>Ref1</div>
        <div ref={e => this.node2 = e}>Ref2</div>
        <button onClick={e => this.btnClick(e)}>button</button>
        <Child ref={this.child}></Child>
        <ChildFunction ref={this.functionChild} message='hello child function'></ChildFunction>
      </div>
    )
  }
}

export default App
```

子组件

```jsx
import React, { PureComponent } from 'react'

export class Child extends PureComponent {
  test() {
    console.log('child test function execution!')
  }
  
  render() {
    return (
      <h2>Child</h2>
    )
  }
}

export default Child
```

函数子组件

```jsx
import { forwardRef } from "react"

const ChildFunction = forwardRef(function (props, ref) {
  console.log('child function props', props)
  return (
    <div>
      <h2>Child FUnction</h2>
      <p ref={ref}>child function ref</p>
    </div>
  )
})

export default ChildFunction
```



### 受控组件

```jsx
import React, { PureComponent } from 'react'

export class App extends PureComponent {

  constructor() {
    super()
    this.state = {
      username: 'zhangsan'
    }
  }

  inputChange(e) {
    console.log(e.target.value)
    this.setState({
      username: e.target.value
    })
  }

  render() {
    const { username } = this.state

    return (
      <div>
        <h1>App</h1>
        <p>{username}</p>
        {/* 受控组件，当没有绑定 value 时为非受控组件 */}
        <input type="text" value={username} onChange={e => this.inputChange(e)} />
      </div>
    )
  }
}

export default App
```



## 第二站

### 受控组件的表单填写

```jsx
import React, { PureComponent } from 'react'

export class App extends PureComponent {

  constructor() {
    super()
    this.state = {
      username: '',
      password: '',
      hobbies: [
        {id: 'sing', name: 'sing', isChecked: false},
        {id: 'dance', name: 'dance', isChecked: false},
        {id: 'rap', name: 'rap', isChecked: false},
      ],
      fruits: []
    }
  }

  inputChange(e) {
    this.setState({
      [e.target.name]: e.target.value
    })
  }

  submitForm(e) {
    e.preventDefault()
    console.log(this.state.username, this.state.password)
    console.log(this.state.hobbies.filter(hobby => hobby.isChecked).map(hobby => hobby.name))
    console.log(this.state.fruits)
  }

  checkHobby(index) {
    const hobbies = [...this.state.hobbies]
    hobbies[index].isChecked = !hobbies[index].isChecked
    this.setState({
      hobbies: hobbies
    })
  }

  selectFruit(e) {
    const fruits = Array.from(e.target.selectedOptions).map(fruit => fruit.value)

    this.setState({
      fruits: fruits
    })
  }

  render() {
    const { username, password, hobbies, fruits } = this.state

    return (
      <div>
        <h1>App</h1>
        <form onSubmit={e => this.submitForm(e)}>
          {/* 受控组件，当没有绑定 value 时为非受控组件 */}
          <label htmlFor="username">
            username: 
            <input id="username" name='username' type="text" value={username} onChange={e => this.inputChange(e)} />
          </label>
          <label htmlFor="password">
            password: 
            <input id="password" name='password' type="password" value={password} onChange={e => this.inputChange(e)} />
          </label>

          <div>
            {
              hobbies.map((hobby, index) => {
                return (
                  <label htmlFor={hobby.id} key={hobby.name}>
                    {hobby.name}: 
                    <input id={hobby.id} type='checkbox'
                          checked={hobby.isChecked}
                          onChange={e => {this.checkHobby(index)}}/>
                  </label>
                )
              })
            }
          </div>

          <div>
            <select value={fruits} multiple onChange={e => this.selectFruit(e)}>
              <option value="apple">Apple</option>
              <option value="orange">Orange</option>
              <option value="banana">Banana</option>
            </select>
          </div>

          <input type='submit' />
        </form>
      </div>
    )
  }
}

export default App
```

### 非受控组件的表单

```jsx
import React, { createRef, PureComponent } from 'react'

export class App extends PureComponent {

  constructor() {
    super()
    this.state = {
      notControlled: 'default value'
    }
    this.notControlledComponent = createRef()
  }

  submitForm(e) {
    e.preventDefault() console.log(this.notControlledComponent.current.value)
  }


  render() {
    const { notControlled } = this.state

    return (
      <div>
        <h1>App</h1>
        <form onSubmit={e => this.submitForm(e)}>
          <label htmlFor="not-controlled">
            not controlled component:
            <input id='not-controlled' type="text" ref={this.notControlledComponent} defaultValue={notControlled}/>
          </label>

          <input type='submit' />
        </form>
      </div>
    )
  }
}

export default App
```

### 高阶组件

高阶组件的英文为 Higher-Order Components，简称为 HOC

高阶组件的参数为组件，返回值为新组件的函数

也即，高阶组件本身不是一个组件而是一个函数，这个函数的参是一个组件，返回值也是一个组件

```js
function HOC(Component) {
    return NewComponent	
}
```

高阶组件函数通过上下文为使用该高阶组件的组件传递上下文

```jsx
import ThemeContext from '../context/ThemeContext'

export default function (Component) {
  return props => (
    <ThemeContext.Consumer>
      {
        value => {
          return (
            <Component {...value} {...props}/>
          )
        }
      }
    </ThemeContext.Consumer>
  )
}
```

父组件通过上下文对象传值

```jsx
import React, { PureComponent } from 'react'
import ThemeContext from './context/ThemeContext'
import About from './components/About'

export class App extends PureComponent {

  render() {
    return (
      <div>
        <h1>App</h1>
        <ThemeContext.Provider value={{color: 'red', backgroundColor: '#eee'}}>
          <About></About>
        </ThemeContext.Provider>
      </div>
    )
  }
}

export default App
```

子组件中的使用上下文传来的值

```jsx
import React, { PureComponent } from 'react'
import WithContext from '../hoc/WithContext'

export class About extends PureComponent {
  render() {
    return (
      <div>
        <h2>About: {this.props.color}-{this.props.backgroundColor}</h2>
      </div>
    )
  }
}

export default WithContext(About)
```

#### 高阶函数的意义

早期 React 提供组件之间一种复用方式时 mixin，目前已经不再使用

- Mixin 可能会相互依赖，相互耦合，不利于代码维护
- 不同的 Mixin 中的方法可能会互相冲突
  - MIxin 非常多时，组件处理起来会比较麻烦，甚至还要为其做相关处理，这样会给代码造成滚雪球式的复杂性

HOC 的缺陷：

- HOC 需要在原组件上进行包裹或者嵌套，如果大量使用 HOC，将会产生非常多的嵌套，这让调试变得非常困难
- HOC 可以劫持 props，在不遵守约定的请胯下也可能造成冲突

Hooks 的出现，是开创性的，它解决了很多的 React 之前存在的问题：

- this 指向问题
- hoc 嵌套复杂度问题



### Poral

通过使用 createPortal 函数可以设置元素绑定到别的 dom 上

自定义 modal 通过 portal 渲染到别的 dom 元素上

```jsx
import React, { PureComponent } from 'react'
import { createPortal } from 'react-dom'

export class Modal extends PureComponent {
  render() {
    return (
      <div>
        {
          createPortal(this.props.children, document.querySelector('#app'))
        }
      </div>
    )
  }
}

export default Modal
```

在组件中使用该组件

```jsx
import React, { PureComponent } from 'react'
import { createPortal } from 'react-dom'
import Modal from './components/Modal'


export class App extends PureComponent {
  render() {
    return (
      <div>
        <h1>App</h1>
        {
          // 绑定到 app dom 上
          createPortal(<h2>App Portal</h2>, document.querySelector('#app'))
        }
        <Modal>
          <h2>Modal Portal Children 1</h2>
          <h3>Modal Portal Children 2</h3>
        </Modal>
      </div>
    )
  }
}

export default App
```



### Fragment

由于 react 最外层必须为一个标签，导致页面渲染之后多使用一个div，可以通过 Fragment 组件，此时界面将不会多渲染一个标签

```jsx
import React, { Fragment, PureComponent } from 'react'


export class App extends PureComponent {
  constructor() {
    super()
    this.state = {
      list: ['name', 'age']
    }
  }
  render() {
    const {list} = this.state
    return (
      // <Fragment>
      //   <h1>App</h1>
      //   <h2>App 2</h2>
      // </Fragment>

      // 语法糖写法
      // <>
      //   <h1>App</h1>
      //   <h2>App 2</h2>
      // </>
      <>
        {
          // 此时必须要写 Fragment，不能使用语法糖的格式
          list.map(item => {
            return (
              <Fragment key={item}>
                <li>{item}</li>
              </Fragment>
            )
          })
        }
      </>
    )
  }
}

export default App
```



### StrictMode

StrictMode 是一个用来突出显示应用程序中潜在问题的工具

- 与 Fragment 一样，StrictMode 不会渲染任何可见的 UI
- 它为后代元素触发额外的检查和警告，严格模式下将会识别过时的方法和类并抛出警告信息
  - 识别不安全的声明周期
  - 使用过时的 ref API
  - 检测意外的副作用
    - 这个组件的 constructor 会被调用两次
    - 这是严格模式下故意进行的操作，让你来检查看在这里写的一些逻辑代码被调用多次时，是否会产生一些副作用
    - 在生产环境中，是不会调用两次的
  - 使用废弃的 findDOMNode 方法
    - 在之前的 React API 中，可以通过 findDOMNode 来获取 DOM，不过已经不推荐使用了
  - 检测过时的 context API
    - 早期的 Context 是通过 static 属性声明 Context 对象属性，通过 getChildContext 返回 Context 对象等方式来使用 Context 的，目前这种方式已经不推荐使用
- 严格模式检查仅在开发模式下运行；它们不会影响生产构建



### 动画

React 通过使用动画插件 react-transition-group 实现组件的入场和离场动画，使用时需要额外的安装

```bash
npm install react-transition-group --save
```

#### CSSTransition

CSSTransition 是基于 Transition 组件构建的

CSSTransition 执行过程中，有三个状态：appear、enter、exit

他们有三种状态，需要定义对应的 CSS 样式

1. 第一类，开始状态：对应的类是 -appear、-enter、exit
2. 第二类：执行动画：对应的类是 -appear-active、-enter-active、-exit-active
3. 第三类：执行结束：对应的类是 -appear-done、-enter-done、-exit-done

CSSTransition 常见对应属性

- in：触发进入或者推出状态
  - 如果添加了 `unmountOnExit={true}` ，那么该组件会在执行退出动画时移除掉
  - 当 in 为 trye 是，触发进入状态，会添加 -enter、-enter-active 的 class 开始执行动画，当动画执行结束后，会移除两个 class、并且添加 -enter-done 的 class
  - 当 in 为 false 时，触发退出状态，会添加 -exit、-exit-active 的 class 开始执行动画，当动画执行结束后，会移除两个 class，并且添加 -enter-done 的 class
- classNames：动画 class 的名称
  - 决定了在编写 css 时，对应的 class 名称：比如 example-enter、example-enter-active、example-enter-done
- timeout：类添加或移除的时间
  - 当该值与样式中设置动画过度时间冲突时，动画执行时间为样式中所设定的，类绑定与消除为该 timeout 属性的值
- appear：是否在初次进入添加动画（需要和 in 同时为 true）
- unmountOnExit：退出后卸载组件

CSSTransition 对应的钩子函数：主要为了检测动画执行过程

- onEnter：在进入动画之前被触发
- onEntering：在应用进入动画时被触发
- onEntered：在应用进入动画结束后被触发

*使用 CSSTransition 在 React 中将会报错：Transition.js:292  Uncaught TypeError: l.findDOMNode is not a function， 因此需要指定该组件的 nodeRef 属性为动画元素的根节点* 

```jsx
import React, { createRef, Fragment, PureComponent } from 'react'
import { CSSTransition } from 'react-transition-group'
import './style/App'

export class App extends PureComponent {
  constructor() {
    super()
    this.state = {
      isActivate: false
    }
    this.nodeRef = createRef()
  }

  render() {
    const { isActivate } = this.state
    
    return (
      <Fragment>
        <button onClick={e => this.setState({isActivate: !isActivate})}>Toggle</button>
        <CSSTransition in={isActivate} timeout={2000} classNames='dh' unmountOnExit={true}
                       // 当不添加 nodeRef 时将会报错，因为该组件使用了 findDOMNode 方法，该方法已经不支持
                       // 通过设置 nodeRef 该组件将会使用 nodeRef 的值作为动画的根节点
                       nodeRef={this.nodeRef} 
                       onEnter={e => console.log('开始进入动画')}
                       onEntering={e => console.log('执行进入动画')}
                       onEntered={e => console.log('执行进入动画结束')}
                       onExit={e => console.log('开始离开动画')}
                       onExiting={e => console.log('执行离开动画')}
                       onExited={e => console.log('执行离开动画结束')}
        >
          {/* 必须存在一个根元素 */}
          <div ref={this.nodeRef}>
            <div>hidden element</div>
            <p>hidden paragraph</p>
          </div>
        </CSSTransition>
      </Fragment>
    )
  }
}

export default App
```

对应的样式

```css
.dh-enter {
  opacity: 0;
}

.dh-enter-active {
  opacity: 1;
  transition: opacity 2s;
}

.dh-exit {
  opacity: 1;
}

.dh-exit-active {
  transition: opacity 2s;
  opacity: 0;
}
```

#### SwitchTransition

SwitchTransition 可以完成两个组件之间切换的炫酷动画

- 比如我们有一个按钮需要在 on 和 off 之间切换，我们希望看到 on 先从左侧退出，off 再从右侧进入
- 这个动画在 vue 中被称之为 vue transition modes
- react-transition-group 中使用 SwitchTransition 来实现该动画

SwitchTransition 中主要有一个属性：mode，有两个值

- in-out：表示新组件先进入，旧组件再移除
- out-in：表示旧组件先移除，新组件再进入

SwitchTransition 组件里面要有 CSSTransition 或者 Transition 组件，其 CSSTransition/Transition 需要设置 key 属性，表示切换不同的状态，该属性值主要设置不一样就代表进行切换

```jsx
import React, { createRef, Fragment, PureComponent } from 'react'
import { CSSTransition, SwitchTransition } from 'react-transition-group'
import './style/App-Transition'

export class App extends PureComponent {
  constructor() {
    super()
    this.state = {
      isLogin: false
    }
    this.nodeRef = createRef()
  }

  render() {
    const { isLogin } = this.state
    
    return (
      <Fragment>
        <SwitchTransition mode='out-in'>
          <CSSTransition nodeRef={this.nodeRef} 
                         key={isLogin ? 'login' : 'logout'}
                         timeout={1000} classNames='dh'>
            <button onClick={e => this.setState({isLogin: !isLogin})}
                    ref={this.nodeRef}>
                      { isLogin ? 'Login' : 'Logout' }
            </button>
          </CSSTransition>
        </SwitchTransition>
      </Fragment>
    )
  }
}

export default App
```

样式设置

```css
.dh-enter {
  transform: translateX(-100%);
}

.dh-enter-active {
  transform: translateX(0);
  transition: transform 1s;
}

.dh-exit {
  transform: translateX(0);
}

.dh-exit-active {
  transition: transform 1s;
  transform: translateX(-100%);
}
```

#### TransitionGroup

给一组元素设置动画

```jsx
import React, { createRef, Fragment, PureComponent } from 'react'
import { TransitionGroup, CSSTransition } from 'react-transition-group'
import './style/APP-TransitionGroup'

export class App extends PureComponent {
  constructor() {
    super()
    this.state = {
      books: [
        {id: 1, name: 'aaa', price: 80},
        {id: 2, name: 'bbb', price: 81},
        {id: 3, name: 'ccc', price: 82},
        {id: 4, name: 'ddd', price: 83},
      ]
    }
    this.nodeRef = []
    for (let i = 0; i < this.state.books.length; ++ i) {
      this.nodeRef.push(createRef())
    }
  }

  addNewBook() {
    const books = [...this.state.books]
    books.push({
      id: new Date().getTime(),
      name: 'eee',
      price: 180
    })
    this.setState({
      books
    })
    this.nodeRef.push(createRef())
  }

  deleteBook(index) {
    const books = [...this.state.books]
    books.splice(index, 1)
    this.setState({ books })
    this.nodeRef.splice(index, 1)
  }

  render() {
    const { books } = this.state
    
    return (
      <Fragment>
        <TransitionGroup>
          {
            books.map((book, index) => {
              return (
                <CSSTransition key={book.id} nodeRef={this.nodeRef[index]}
                               timeout={1000} classNames='example' unmountOnExit={true}>
                  <li ref={this.nodeRef[index]}>
                    <span>{book.name}-{book.price}</span>
                    <button onClick={e => this.deleteBook(index)}>delete</button>
                  </li>
                </CSSTransition>
              )
            })
          }
        </TransitionGroup>
        <button onClick={e => this.addNewBook()}>Add new book</button>
      </Fragment>
    )
  }
}

export default App
```

CSS 文件

```css
.example-enter {
  transform: translateX(100px);
}

.example-enter-active {
  transform: translateX(0);
  transition: transform 1s ease;
}

.example-exit {
  transition: transform 1s ease;
  transform: translateX(0);
}

.example-exit-active {
  transform: translateX(100px);
}
```



### 样式

#### 内联样式

style 接受一个采用小驼峰命名属性的 JavaScript 对象，并且可以引用 state 中的状态来设置相关的样式

内联样式的优点：

1. 内联样式，样式之间不会有冲突
2. 可以动态获取当前 state 中的状态

内联样式的缺点：

1. 写法上都需要使用驼峰表示
2. 某些样式没有提示
3. 大量的样式，代码混乱
4. 某些样式无法编写（比如伪类，伪元素）

```jsx
import React, { PureComponent } from 'react'

export class App extends PureComponent {

  render() {
    const style = {
      color: 'red',
      fontSize: '20px',
      border: '1px solid #eee'
    }
    return (
      <div>
        <h1 style={style}>App</h1>
      </div>
    )
  }
}

export default App
```

#### 引用外部 CSS 文件

该方式将会对所有文件生效，即使是子组件中引用的样式文件也将对所有文件生效

父组件

```jsx
import React, { PureComponent } from 'react'
import Home from './components/Home'

export class App extends PureComponent {

  render() {
    return (
      <div>
        <h1 className='title'>App</h1>
        <Home></Home>
      </div>
    )
  }
}

export default App
```

子组件

```jsx
import React, { PureComponent } from 'react'
import '../styles/App'

export class Home extends PureComponent {
  render() {
    return (
      <div>
        <h2 className='title'>Home</h2>
      </div>
    )
  }
}

export default Home
```

样式文件（此样式将对所有文件生效）

```css
.title {
  color: red;
  border: 1px solid #eee;
}
```

#### CSS Modules

css modules 并不是 React 特有的解决方案，而是所有使用了类似于 webpack 配置的环境下都可以使用的

- 如果在其他项目中使用它，那么我们需要自己来进行配置，比如配置 webpack.config.js 中的 modules: true 等

React 的脚手架已经内置了 css modules 的配置

- .css/.less/.scss 等样式文件都需要修改成 .module.css/.module.less/.module.scss 等

css modules 确实解决了局部作用域的问题，但是这种方案仍存在如下的缺陷：

1. 引用的类名，不能使用连接符（.home-title）其中 - 在 JavaScript 不被识别
2. 所有的 className 都必须使用 {style.className} 的形式来编写
3. 不方便动态修改某些样式，仍然需要使用内联样式的方式

配置 webpack.config.js

```js
const path = require('path')
const HTMLPlugin = require('html-webpack-plugin')
const {ProvidePlugin} = require('webpack')
const TerserPlugin = require('terser-webpack-plugin')

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', {
          loader: 'css-loader',
          options: {
            modules: {
              // 使用 named 进行导出（即 export 方式），通过使用 import {类名} 导入
              namedExport: true,
              // 设置类的名字格式
              localIdentName: '[local]_[contenthash:6]'
            }
          }
        }]
      },
    ]
  }
}
```

在组件中使用该样式

```jsx
import React, { PureComponent } from 'react'
import {title} from './styles/App'

export class App extends PureComponent {

  render() {
    console.log(title)
    return (
      <div>
        <h1 className={title}>App</h1>
      </div>
    )
  }
}

export default App
```

#### CSS in JS

CSS-in-JS 是指一种模式，其中 CSS 由 JavaScript 生成，而不是在外部中定义

这个功能并不是 React 的一部分，而是由第三方库提供

- CSS-in-JS 通过 JavaScript 来为 CSS 赋予一些能力，包括 类似于 CSS 预处理器一样的样式嵌套、函数定义、逻辑复用、动态修改状态等
- 虽然 CSS 预处理器也具备某些能力，但是获取动态状态依然是一个不好处理的点

目前比较理性的 CSS-in-JS 的库：

1. styled-components（常用）
2. emotion
3. glamorous

使用 styled-components

```bash
npm install styled-components -D
```

定义 CSS js 文件

```js
import styled from "styled-components"

// 可以通过 attrs 函数设置样式属性
const AppStyleWrapper = styled.div.attrs(props => ({
  color: props.color || 'red',
  backgroundColor: '#ff8800'
}))`
  .title {
    color: ${props => props.color};
    font-size: ${props => props.fontSize};
    border: 1px solid #aaa;

    &:hover {
      background-color: ${props => props.backgroundColor};
      color: #fff;
    }
  }
`

export default AppStyleWrapper
```

在组件中使用该样式

```jsx
import React, { PureComponent } from 'react'
import AppStyleWrapper from './styles/App'

export class App extends PureComponent {

  render() {
    console.log(AppStyleWrapper)
    return (
      // 传递参数，该组件将参数接收到 props 中
      <AppStyleWrapper color='yellow' fontSize='50px'>
        <h1 className='title'>App</h1>
      </AppStyleWrapper>
    )
  }
}

export default App
```

#### 主题和继承

About 组件样式

```js
import styled from "styled-components"

const AboutFatherStyleWrapper = styled.div`
  .content {
    border: 1px solid #aaa;
  }
`

// 继承 AboutFatherStyleWrapper 样式
const AboutStyleWrapper = styled(AboutFatherStyleWrapper)`
  .about-title {
    color: ${props => props.theme.color};
    font-size: 18px;
  }

  .content {
    color: ${props => props.theme.color};
    background-color: ${props => props.theme.backgroundColor};
  }
`

export default AboutStyleWrapper
```

About 组件

```jsx
import React, { PureComponent } from 'react'
import AboutStyleWrapper from './style'

export class About extends PureComponent {
  render() {
    return (
      <AboutStyleWrapper>
        <h2 className='about-title'>About</h2>
        <div className='content'>About content</div>
      </AboutStyleWrapper>
    )
  }
}

export default About
```

App 组件

```jsx
import React, { PureComponent } from 'react'
import {ThemeProvider} from 'styled-components'
import AppStyleWrapper from './styles/App'
import About from './components/About'


export class App extends PureComponent {

  render() {
    console.log(AppStyleWrapper)
    return (
      // 通过 ThemeProvider 向所有子组件提供主题样式
      <ThemeProvider theme={{color: '#909090', backgroundColor: '#09aaf3'}}>
        {/* 传递参数，该组件将参数接收到 props 中 */}
        <AppStyleWrapper fontSize='50px'>
          <h1 className='title'>App</h1>
        </AppStyleWrapper>
        <About></About>
      </ThemeProvider>
    )
  }
}

export default App
```

### classnames 库

在需要动态添加样式时，使用 react 会比较繁琐

```jsx
import React, { Fragment, PureComponent } from 'react'


export class App extends PureComponent {
  constructor() {
    super()
    this.state = {
      isActive: true,
      isDelete: true
    }
  }

  render() {
    const { isActive, isDelete } = this.state
    return (
      <Fragment>
        <div className={`${isActive? 'active' : ''} ${isDelete ? 'delete' : ''}`}>App</div>
      </Fragment>
    )
  }
}

export default App
```

通过使用 classnames 库来简化动态类的添加

```bash
npm install classnames
```

在组件中的使用

```jsx
import React, { Fragment, PureComponent } from 'react'
import classnames from 'classnames'

export class App extends PureComponent {
  constructor() {
    super()
    this.state = {
      isActive: true,
      isDelete: true
    }
  }

  render() {
    const { isActive, isDelete } = this.state
    return (
      <Fragment>
        <div className={classnames('title', {'active': isActive, 'delete': isDelete})}>App</div>
        <div className={classnames(['title', {'active': isActive, 'delete': isDelete}])}>App</div>
      </Fragment>
    )
  }
}

export default App
```



## 第三站

### 纯函数

在程序设计中，若一个函数符合以下条件，那么这个函数就是纯函数

1. 此函数在相同的输入值时，需要产生相同的输出
2. 函数的输出和输入值以外的其他隐藏信息或状态无关
3. 该函数不能由语义上可观察的函数副作用，诸如 触发事件，使用输入输出设备，或更改输出值以外的内容等
   1. 副作用表示在执行一个函数时，除了返回数值以外，还对调用函数产生了附加的影响，比如修改了全局变量，修改参数或者改变外部的存储

纯函数的优点：

1. 在编写纯函数的时候，只需要单纯实现自己的业务逻辑即可不需要关心传入的内容是如何获得的或者依赖其他的外部变量是否已经发生了修改
2. 在使用的时候，只需要确保输入的内容不会被任意篡改，并且确定的输入一定会有确定的输出



### Redux

随着需要管理的状态越来越多，越来越复杂，以及要管理的状态（包括服务器返回的数据，缓存数据、用户操作产生的数据、UI的状态），需要对状态进行监管。

React 是在视图层帮助我们解决了 DOM 的渲染过程，但是 State 任然留给我们自己管理

- 无论是组件定义自己的 state，还是组件之间的通信通过 props 进行传递；也包括 context 进行数据之间的共享
- React 主要负责帮助我们管理视图，state 如何维护最终还是我们自己来决定

Redux 就是一个帮助我们管理 state 的容器；Redux 是 JavaScript 的状态容器，提供了可预测的状态管理

Redux 除了可以和 React 一起使用之外，它也可以和其他界面库一起来用（Vue）并且他很小（包括依赖在内，只有 2kb）

- Redux要求我们通过 action 来更新数据
  - 所有数据的变化，必须通过派发（dispath）action 来更新
  - action 是一个普通的 JavaScript 对象，用来描述这次更新 type 和 content
  - 强制使用 action 的好处是可以清晰知道数据到底发生了什么样的变化，所有数据都是可追踪、可预测的
- state 和 action 通过 reducer 进行联系
  - reducer 是一个纯函数
  - reducer 做的事情就是将传入的 state 和 action 结合起来生成一个新的 state

引入 redux 包

```bash
npm install redux --save
```

redux 基本使用

```js
const {createStore} = require('redux')

const data = {
  username: 'zhangsan',
  age: 16
}

/**
 * 该函数将会被执行两次，第一次是刚开始创建的时候
 * @param {Object} state 未被修改的 state 的值
 * @param {Object} action 使用 store.dispatch 传入的 action
 * @returns 返回值将作为 store 之后存储的 state
 */
function reducer(state = data, action) {
  // 第一次打印：reducer:  { username: 'zhangsan', age: 16 } { type: '@@redux/INIT7.d.i.y.a.5' }
  // 第二次打印：reducer:  { username: 'zhangsan', age: 16 } { type: 'update_username', username: 'wangwu' }
  console.log('reducer: ', state, action)

  if (action.type == 'update_username') {
    return { ...state, username: action.username }
  }

  return state
}

const store = createStore(reducer)

console.log(store.getState()) // { username: 'zhangsan', age: 16 }
// 这样修改将导致界面不会发生动态更新
// store.getState().username = 'lisi'
// console.log(store.getState())

const res = store.dispatch({type: 'update_username', username: 'wangwu'})
console.log(res) // { type: 'update_username', username: 'wangwu' }
console.log(store.getState()) // { username: 'wangwu', age: 16 }
```

通过订阅模式进行事件监听

```js
const { createStore } = require('redux')

const data = [
  {name: 'article1', author: 'zhangsan'},
  {name: 'article2', author: 'lisi'}
]

function reducer(state = data, action) {
  switch(action.type) {
    case 'public article': 
      return [...state, action.article]
    case 'delete article':
      let res = [...state]
      res.splice(action.start, action.count)
      return res
    default:
      return state
  }
}

const store = createStore(reducer)
const unsubscribe = store.subscribe(() => {
  console.log('store subscribe: ', store.getState())
})

const publicAction = (article) => ({
  type: 'public article',
  article
})

const deleteAction = (start, count) => ({
  type: 'delete article',
  start: start,
  count: count
})

/*
store subscribe:  [
  { name: 'article1', author: 'zhangsan' },
  { name: 'article2', author: 'lisi' },
  { name: 'a1', author: 'wangwu' }
]
*/
store.dispatch(publicAction({name: 'a1', author: 'wangwu'}))
/*
store subscribe:  [
  { name: 'article1', author: 'zhangsan' },
  { name: 'a1', author: 'wangwu' }
]
*/
store.dispatch(deleteAction(1, 1))
unsubscribe()
// 下面将不会触发订阅事件
store.dispatch(publicAction({name: 'a3', author: 'laowang'}))
```

对使用过程进行优化，通常将文件分为如下四个部分：

```bash
actionCreator.js # 存放创建的 action 函数
constants.js # 存放常量，即 switch 中的 case
index.js # 存放 store 对象
reducer.js # 存放数据以及 reducer 函数
```

actionCreator.js

```js
import { CHANGE_AGE, CHANGE_NAME } from "./constants";

export function changeName(username) {
  return {
    type: CHANGE_NAME,
    username: username
  }
}

export function changeAge(age) {
  return {
    type: CHANGE_AGE,
    age: age
  }
}
```

constants.js

```js
export const CHANGE_NAME = 'CHANGE_NAME'
export const CHANGE_AGE = 'CHANGE_AGE'
```

index.js

```js
import { createStore } from "redux"

import {reducer} from './reducer'

const store = createStore(reducer)

export default store
```

reducer.js

```js
import { CHANGE_AGE, CHANGE_NAME } from "./constants"

const data = {
  username: 'zhangsan',
  age: 18
}

function reducer(state = data, action) {
  switch(action.type) {
    case CHANGE_NAME:
      return {...state, username: action.username}
    case CHANGE_AGE:
      return {...state, age: action.age}
    default:
      return state
  }
}

export default reducer
```

#### Redux 三大原则

单一数据源

- 整个应用程序的 state 被存储在一颗 object tree 中，并且 这个 object tree 只存储在一个 store 中
- Redux 并没有强制让我们不能创建多个 Store，但是那样做并不利于数据的维护
- 单一的数据源可以让整个应用程序的 state 变得方便维护、追踪、修改

State 是只读的

- 唯一修改 state 的方法一定是触发 action，不要试图在其他地方通过任何的方式修改 state
- 这样就确保了 View 或网络请求都不能直接修改 state，他们只能通过 action 来描述自己想要如何修改 state
- 这样可以保证所有的修改都被集中化处理，并且按照严格的顺序来执行，所以不需要担心 race condition（竞态）的问题

使用纯函数来执行修改

- 通过 reducer 将 旧 state 和 actions 联系在一起，并且返回一个新的 state
- 随着应用程序的复杂度增加，我们可以将 reducer 拆分成多个小的 reducer 分别操作不同的 state tree 的一部分
- 但是所有的 reducer 都应该是纯函数，不能产生任何的副作用



### 在 React 中使用 Redux

```jsx
import React, { PureComponent } from 'react'
import store from './store/system'
import { addCount, desCount } from './store/system/actionCreators'
import About from './components/About'

export class App extends PureComponent {
  constructor() {
    super()
    this.state = {
      count: store.getState().count
    }
  }

  componentDidMount() {
    store.subscribe(() => {
      this.setState({
        count: store.getState().count
      })
    })
  }

  render() {
    const {count} = this.state

    return (
      <div>
        <h1>App</h1>
        <div>{count}</div>
        <button onClick={e => store.dispatch(addCount(1))}>Add</button>
        <button onClick={e => store.dispatch(desCount(1))}>Des</button>
        <About></About>
      </div>
    )
  }
}

export default App
```

#### React-Redux

在 react 中使用封装后的 redux

```bash
npm install react-redux
```

在根组件中使用 Provider 给后续组件设置 store

```jsx
import {createRoot} from 'react-dom/client'
import { StrictMode } from 'react'
import { Provider } from "react-redux"
import store from './store'

import App from "./App"

const root = createRoot(document.querySelector('#root'))
root.render(
  <StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </StrictMode>
)
```

在需要使用 store 组件中使用 connect 进行使用

```jsx
import React, { PureComponent } from 'react'
import { connect } from 'react-redux'
import store from '../store'
import { changeUsername } from '../store/actionCreator'

export class About extends PureComponent {
  changeUsername() {
    this.props.changeUsername('lisi')
    console.log(this.props, store.getState())
  }

  render() {
    const { username } = this.props
    console.log(this.props, store.getState())
    return (
      <div>
        <h2>About</h2>
        <p>{username}</p>
        <button onClick={() => this.changeUsername()}>Change Username</button>
      </div>
    )
  }
}

// 过滤不需要的数据
function mapStateToProps(state) {
  console.log(state)
  return {
    username: state?.username,
    age: state?.age
  }
}

function mapDispatchToProps(dispatch) {
  return {
    changeUsername(username) {
      dispatch(changeUsername(username))
    }
  }
}
// connect 函数执行后返回一个高阶组件函数
// connect 接收两个函数作为参数
export default connect(mapStateToProps, mapDispatchToProps)(About)
```

当使用异步请求时，需要对 dispatch 传递一个函数作为分发的对象，此时需要对 redux 进行增强，安装 redux-thunk 库

```bash
npm install redux-thunk
```

在 actionCreator.js 文件中编写函数

```js
export function addUser(userList) {
  return {
    type: ADD_USERS,
    users: userList
  }
}

export function fetchData() {
  // 定义异步请求函数，将该函数进行返回
  function _fetch(dispatch, getState) {
    fetch('http://jsonplaceholder.typicode.com/posts', {method: 'get'}).then(async res => {
      const userList = await res.json()
      dispatch(addUser(userList))
    })
  }

  return _fetch
}
```

在组件中只需要调用该函数即可

```jsx
import React, { PureComponent } from 'react'
import { connect } from 'react-redux'
import { fetchData } from '../store/actionCreator'

export class About extends PureComponent {

  componentDidMount() {
    this.props.fetchData()
  }

  render() {
    const { users } = this.props
    return (
      <div>
        <h2>About</h2>
        <ul>
          {
            users?.map(user => (
              <li key={user.id}>{user.id} - {user.userId} - {user.title}</li>
            ))
          }
        </ul>
      </div>
    )
  }
}

// 过滤不需要的数据
function mapStateToProps(state) {
  return {
    users: state?.users
  }
}

function mapDispatchToProps(dispatch) {
  return {
    fetchData() {
      dispatch(fetchData())
    }
  }
}
// connect 函数执行后返回一个高阶组件函数
// connect 接收两个函数作为参数
export default connect(mapStateToProps, mapDispatchToProps)(About)
```

redux dev tools 开启

```js
import { legacy_createStore as createStore, applyMiddleware, compose } from 'redux'
import { thunk } from 'redux-thunk'

import reducer from './reducer'

// 传递 trace 设置开启调用栈
const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({trace: true}) || compose

// 开启对 redux 扩展的支持
const store = createStore(reducer, composeEnhancers(applyMiddleware(thunk)))

export default store
```

#### Redux 拆分

通过创建独立的文件夹区分不同的 redux，在 index.js 中合并不同的  reducer 

```js
import { legacy_createStore as createStore, applyMiddleware, compose, combineReducers } from 'redux'
import { thunk } from 'redux-thunk'

import homeReducer from "../home/reducer"
import profileReducer from "../profile/reducer"

const reducer = combineReducers({
  home: homeReducer,
  profile: profileReducer
})

// 传递 trace 设置开启调用栈
const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({trace: true}) || compose

// 开启对 redux 扩展的支持
const store = createStore(reducer, composeEnhancers(applyMiddleware(thunk)))

export default store
```

combineReducers 函数的实现类似于下面

```js
function reducer (state = {}, action) {
  // 初始时，传入 reducer 的参数为：state: undefined action: {type: '@@INIT'}
  console.log(state, action)
  
  return {
    home: homeReducer(state.home, action),
    profile: profileReducer(state.profile, action)
  }
}
```

#### Redux 插件

```js
import { legacy_createStore as createStore, applyMiddleware, compose } from 'redux'
import { thunk } from 'redux-thunk'

import reducer from './reducer'

// 传递 trace 设置开启调用栈
const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({trace: true}) || compose

// 开启对 redux 扩展的支持
const store = createStore(reducer, composeEnhancers(applyMiddleware(thunk)))

// 记录日志，通过对 dispatch 进行拦截
function log(store) {
  const dispatch = store.dispatch
  function recordLog(action) {
    console.log('before action execute', action)
    dispatch(action)
  }
  store.dispatch = recordLog
}
log(store)

export default store
```

#### Redux Toolkit

redux toolkit 是官方推荐编写 redux 逻辑的方法，该工具能解决使用 redux 编写时逻辑过于繁琐，代码量过多且不利于管理等问题

redux toolkit 的核心 API 主要包含如下：

- configureStore：包装 createStore 以提供简化的配置选项和良好的默认值，它可以自动组合你的 slice reducer，添加你提供的任何 redux 中间件，redux-thunk 默认包含，并启用 redux devtool extension
- createSlice：接受 reducer 函数的对象，切片名称和初始状态值，并自动生成切片 reducer，并带有相应的 actions
- createAsyncThunk：接受一个动作类型字符串和一个返回 Promise 的函数，并生成一个 pending/fulfilled/rejected 基于该 Promise分派动作类型的 thunk

安装 redux-toolkit

```bash
npm install @reduxjs/toolkit react-redux
```

创建 store/index.js

```js
import { configureStore } from '@reduxjs/toolkit'
import profileReducer from './features/profile'

const store = configureStore({
  reducer: {
    profile: profileReducer
  }
})

export default store
```

创建对应的 reducer

```js
import { createSlice } from "@reduxjs/toolkit"

const profileSlice = createSlice({
  name: 'profile',
  initialState: {
    username: 'zhangsan',
    age: 18
  },
  reducers: {
    // action: {type: 'profile/changeUsername', payload: 'lisi'}
    changeUsername(state, action) {
      console.log(state, action)
      state.username = action.payload
    },
    changeAge(state, aciton) {
      console.log(state, aciton)
      state.age = action.payload
    }
  }
})


export const { changeUsername, changeAge } = profileSlice.actions
export default profileSlice.reducer
```

在组件中的使用

```jsx
import React, { PureComponent } from 'react'
import { connect } from 'react-redux'

import { changeUsername, changeAge } from '../store/features/profile'

export class Profile extends PureComponent {
  render() {
    const { username, age } = this.props

    return (
      <div>
        <h2>Profile</h2>
        <p>{username} - {age}</p>
        <button onClick={() => this.props.changeUsername('lisi')}>Change Username</button>
      </div>
    )
  }
}

function mapStateToProps(state) {
  return {
    username: state.profile.username,
    age: state.profile.age
  }
}

function mapDispatchToProps(dispatch) {
  return {
    changeUsername(username) {
      dispatch(changeUsername(username))
    }
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(Profile)
```

在 Redux Toolkit 中使用异步函数

```js
import { createSlice, createAsyncThunk } from "@reduxjs/toolkit"

export const fetchData = createAsyncThunk('fetch/home', async (extractInfo, store) => {
  const res = await fetch('http://jsonplaceholder.typicode.com/posts', {method: 'get'})
  const data = await res.json()
  // 可以通过如下方式直接在这个里面添加数据
  store.dispatch(addUser(data))
  return data
})

const homeSlice = createSlice({
  name: 'home',
  initialState: {
    users: []
  },
  reducers: {
    addUser(state, action) {
      state.users = action.payload
    }
  },
  // 这种方式现已不支持
  // extraReducers: {
  //   [fetchData.pending](state, action) {
  //     console.log(state, action, 'ready to fetch data')
  //   },
  //   [fetchData.fulfilled](state, action) {
  //     console.log(state, action, 'have got data')
  //   },
  //   [fetchData.rejected](state, action) {
  //     console.log(state, action, 'get data error')
  //   }
  // },
  // 支持的写法
  extraReducers: (builder) => {
    builder.addCase(fetchData.pending, (state, action) => {
      console.log(state, action, 'ready to fetch data')
    }).addCase(fetchData.fulfilled, (state, action) => {
      console.log(state, action, 'have got data')
      // state.users = action.payload
    }).addCase(fetchData.rejected, (state, action) => {
      console.log(state, action, 'get data error')
    })
  }
})

export const { addUser } = homeSlice.actions
export default homeSlice.reducer
```

在组件中的使用

```jsx
import React, { PureComponent } from 'react'
import { connect } from 'react-redux'

import { fetchData } from '../store/features/home'

export class Home extends PureComponent {
  componentDidMount() {
    this.props.fetchHomeData()
  }

  render() {
    const { users } = this.props
    
    return (
      <div>
        <h2>Home</h2>
        <ul>
          {
            users.map(user => (
              <li key={user.id}>{user.title}</li>
            ))
          }
        </ul>
      </div>
    )
  }
}

const mapStateToProps = (state) => ({
  users: state.home.users
})
const mapDispatchToProps = (dispatch) => ({
  fetchHomeData() {
    dispatch(fetchData({path: '123'}))
  }
})

export default connect(mapStateToProps, mapDispatchToProps)(Home)
```

#### Redux Toolkit 的数据不可变性

Redux Toolkit 底层使用了 immerjs 的一个库来保证数据的不可变性

为了节约内存，又出现了一个新的算法：Persistent Data Structure （持久化数据结构或一致性数据结构）

- 用一种数据结构来保存数据
- 当数据被修改时，会返回一个对象，但是新的对象会尽可能利用之前的数据结构而不会对内存造成浪费

#### connect 函数的实现

connect 函数

```jsx
import { PureComponent } from "react"
// import store from "../store"
import StoreContext from './StoreContext'

/**
 * realize a connect function
 * @param {Function} mapStateToProps 
 * @param {Function} mapDispatchToProps 
 * @returns high component function
 */
export default function connect(mapStateToProps, mapDispatchToProps) {
  return (Component) => {
    class NewComponent extends PureComponent {
      constructor(props, context) {
        super(props)
        console.log(context)
        this.state = mapStateToProps(context.getState())
      }
      componentDidMount() {
        this.unsubscribe = this.context.subscribe(() => {
          this.setState(mapStateToProps(this.context.getState()))
        })
      }
      componentWillUnmount() {
        this.unsubscribe()
      }
      render() {
        // state dispatch
        const states = mapStateToProps(this.context.getState())
        const dispatchs = mapDispatchToProps(this.context.dispatch)
        return <Component {...this.props} {...states} {...dispatchs}/>
      }
    }

    NewComponent.contextType = StoreContext
    // return newComponent
    return NewComponent
  }
}
```

StoreContext

```js
import { createContext } from "react"

const StoreContext = createContext()

export default StoreContext
```

main.js

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { Provider } from 'react-redux'
import StoreContext from './hoc/StoreContext'

import store from './store'
import App from './App'

const root = createRoot(document.querySelector('#root'))
root.render(
  <StrictMode>
    {/* 必须要加上 Provider */}
    <Provider store={store}>
      <StoreContext.Provider value={store}>
        <App />
      </StoreContext.Provider>
    </Provider>
  </StrictMode>
)
```

#### React 中 state 管理

react 中管理状态可以通过：1. 组件中自己的state、2. Context 数据的共享状态、3. Redux 管理应用状态

- UI 相关的组件内部可以维护的状态，在组件内部自己来维护
- 大部分需要共享的状态，都交给 redux 来管理和维护
- 从服务器请求的数据（包括请求的操作），交给 redux 来维护



### React Router

安装相关依赖

```bash
npm install react-router-dom
```

react-router 会包含一些 react-native 的内容，web 开发并不需要

react-router 最主要的时给我们提供一些组件：

- BrowserRouter 或 HashRouter
  - Router 中包含了对路径改变的监听，并且会将相应的路径传递给子组件
  - BrowserRouter 使用 history 模式
  - HashRouter 使用 hash 模式
- Routes：包裹所有的 Route，在其中匹配一个路由
  - Router5.x 使用的是 Switch 组件
- Route：Route 用于匹配的路径
  - path 属性：用于设置匹配到的路径
  - element 属性：设置匹配到路径后，渲染的组件
    - Router5.x 使用的是 component 属性
  - exact：精准匹配，只有精准匹配到完全一致的路径，才会渲染对应的组件
    - Router6.x 不再支持该属性
- Link 和 NavLink：
  - 通常路径的跳转是使用 Link 组件，最终会被渲染成 a 元素
  - NavLink 是 Link 基础之上增加了一些样式属性
  - to 属性：Link 中最重要的属性，用于设置跳转到的路径

#### Link 基本使用

```jsx
import React, { PureComponent } from 'react'
import { Routes, Route, Link } from 'react-router-dom'

import Home from './pages/Home'
import Profile from './pages/Profile'

export class App extends PureComponent {
  render() {
    return (
      <div>
        <h1>App</h1>
        <div className="header">
          <Link to='/home'>Home</Link>
          <Link to='/profile'>Profile</Link>
          <hr />
        </div>
        <div className="content">
          <Routes>
            <Route path='/home' element={<Home />} />
            <Route path='/profile' element={<Profile />} />
          </Routes>
        </div>
        <div className="footer">
          <hr />
          Footer
        </div>
      </div>
    )
  }
}

export default App
```

#### NavLink 基本使用

- style: 传入一个函数，函数接受一个对象，包含 isActive 属性
- className： 传入函数，函数接受一个对象，包括 isActive 属性
- 默认的 activeClassName：
  - 事实上再默认匹配成功时，NavLink 就会添加上一个动他的 active class，可以直接使用该 class

```jsx
import React, { PureComponent } from 'react'
import { Routes, Route, Link, NavLink } from 'react-router-dom'

import Home from './pages/Home'
import Profile from './pages/Profile'
import BaseStyle from './style/base'

export class App extends PureComponent {
  render() {
    return (
      <BaseStyle>
        <h1>App</h1>
        <div className="header">
          {/* Link 当标签激活时不会添加 class */}
          {/* <Link to='/home'>Home</Link>
          <Link to='/profile'>Profile</Link> */}
          {/* NavLink 再标签激活时会添加 active class */}
          {/* <NavLink to='/home'>Home</NavLink> */}
          {/* 通过修改 className 改变激活时绑定的 class 属性 */}
          <NavLink to='/home' className={({isActive}) => isActive ? 'link-active' : ''}>Home</NavLink>
          {/* <NavLink to='/profile'>Profile</NavLink> */}
          {/* {isActive: true, isPending: false, isTransitioning: false} */}
          <NavLink to='/profile' style={({isActive}) => ({color: isActive ? 'red' : ''})}>Profile</NavLink>
          <hr />
        </div>
        <div className="content">
          <Routes>
            <Route path='/home' element={<Home />} />
            <Route path='/profile' element={<Profile />} />
          </Routes>
        </div>
        <div className="footer">
          <hr />
          Footer
        </div>
      </BaseStyle>
    )
  }
}

export default App
```

#### Navigate 导航

Navigate 用于路由的重定向，当这个组件出现时，就会执行跳转到对应的 to 路径中

```jsx
import React, { PureComponent } from 'react'
import { Routes, Route, Link, NavLink, Navigate } from 'react-router-dom'

import Home from './pages/Home'
import Profile from './pages/Profile'
import BaseStyle from './style/base'
import NotFound from './pages/NotFound'

export class App extends PureComponent {
  render() {
    return (
      <BaseStyle>
        <h1>App</h1>
        <div className="header">
          <Link to='/home'>Home</Link>
          <Link to='/profile'>Profile</Link>
          <hr />
        </div>
        <div className="content">
          <Routes>
            {/* 设置路由自动跳转 */}
            <Route path='/' element={<Navigate to='/home'/>}/>
            <Route path='/home' element={<Home />} />
            <Route path='/profile' element={<Profile />} />
            <Route path='*' element={<NotFound />}/>
          </Routes>
        </div>
        <div className="footer">
          <hr />
          Footer
        </div>
      </BaseStyle>
    )
  }
}

export default App
```

#### 路由嵌套

```jsx
import React, { PureComponent } from 'react'
import { Routes, Route, Link, NavLink, Navigate } from 'react-router-dom'

import Home from './pages/Home'
import Profile from './pages/Profile'
import BaseStyle from './style/base'
import NotFound from './pages/NotFound'

import HomeRecommand from './pages/HomeRecommand'
import HomeRanking from './pages/HomeRanking'

export class App extends PureComponent {
  render() {
    return (
      <BaseStyle>
        <h1>App</h1>
        <div className="header">
          <Link to='/home'>Home</Link>
          <Link to='/profile'>Profile</Link>
          <hr />
        </div>
        <div className="content">
          <Routes>
            {/* 设置路由自动跳转 */}
            <Route path='/' element={<Navigate to='/home'/>}/>
            <Route path='/home' element={<Home />}>
              <Route path='/home' element={<Navigate to='/home/recommand' />}></Route>
              <Route path='/home/recommand' element={<HomeRecommand/>}></Route>
              <Route path='/home/ranking' element={<HomeRanking />}></Route>
            </Route>
            <Route path='/profile' element={<Profile />} />
            <Route path='*' element={<NotFound />}/>
          </Routes>
        </div>
        <div className="footer">
          <hr />
          Footer
        </div>
      </BaseStyle>
    )
  }
}

export default App
```

Home.jsx

```jsx
import React, { PureComponent } from 'react'
import { Link, Outlet } from 'react-router-dom'

export class Home extends PureComponent {
  render() {
    return (
      <div>
        <h2>Home</h2>
        <Link to='/home/recommand'>Recommand</Link>
        <Link to='/home/ranking'>Ranking</Link>
        {/* 设置占位符，渲染到此处 */}
        <Outlet></Outlet>
      </div>
    )
  }
}

export default Home
```

#### useNavigator

```jsx
import React from 'react'
import { Routes, Route, Link, NavLink, Navigate, useNavigate } from 'react-router-dom'

import Home from './pages/Home'
import Profile from './pages/Profile'
import BaseStyle from './style/base'
import NotFound from './pages/NotFound'

import HomeRecommand from './pages/HomeRecommand'
import HomeRanking from './pages/HomeRanking'

export function App(props) {
  // 调用必须再顶层使用，不能在函数中使用
  const navigator = useNavigate()

  return (
    <BaseStyle>
      <h1>App</h1>
      <div className="header">
        <button onClick={e => navigator('/home')}>Home Button</button>
        <button onClick={e => navigator('/profile')}>Profile Button</button>
        <hr />
      </div>
      <div className="content">
        <Routes>
          {/* 设置路由自动跳转 */}
          <Route path='/' element={<Navigate to='/home'/>}/>
          <Route path='/home' element={<Home />}>
            <Route path='/home' element={<Navigate to='/home/recommand' />}></Route>
            <Route path='/home/recommand' element={<HomeRecommand/>}></Route>
            <Route path='/home/ranking' element={<HomeRanking />}></Route>
          </Route>
          <Route path='/profile' element={<Profile />} />
          <Route path='*' element={<NotFound />}/>
        </Routes>
      </div>
      <div className="footer">
        <hr />
        Footer
      </div>
    </BaseStyle>
  )
}

export default App
```

对 navigator 进行包装，使其可以在类组件中使用

```jsx
import { useLocation, useNavigate, useParams, useSearchParams } from "react-router-dom"

function withRouter(Component) {

  return function (props) {
    const navigator = useNavigate()
    // 当通过动态路由传递参数时，使用 params 对象获取
    const params = useParams()
    // 查询字符串的参数
    const location = useLocation()
    const [searchParams] = useSearchParams()
    const router = {navigator, params, location, searchParams}
    return (
      <Component {...props} router={router} />
    )
  }
}

export default withRouter
```

获取传递的值

```jsx
import React, { PureComponent } from 'react'
import withRouter from '../hoc/withRouter'


export class Profile extends PureComponent {
  render() {
    const { params, location, searchParams } = this.props.router
    console.log(location, location.search)
    console.log(searchParams.get('name'), searchParams.get('age'))
    return (
      <div>Profile: {params.id}</div>
    )
  }
}

export default withRouter(Profile)
```

#### router 文件编写

当使用配置去设置 router 时，需要使用 useRoutes hook，并且使用懒加载方式的组件需要被 Suspense 包裹

main.js

```jsx
import { createRoot } from 'react-dom/client'
import { StrictMode, Suspense } from 'react'

import { HashRouter } from 'react-router-dom'

import App from './App'

const root = createRoot(document.querySelector('#root'))

root.render(
  <StrictMode>
    <HashRouter>
      {/* 当路由设置懒加载之后，必须使用 Suspense 包裹 */}
      <Suspense fallback={<h3>Loading ...</h3>}>
        <App />
      </Suspense>
    </HashRouter>
  </StrictMode>
)
```

router.js

```jsx
import { Navigate } from 'react-router-dom'

import Home from '../pages/Home'
import Profile from '../pages/Profile'
import NotFound from '../pages/NotFound'

// import HomeRecommand from '../pages/HomeRecommand'
// import HomeRanking from '../pages/HomeRanking'
import React from 'react'

// 设置懒加载
const HomeRecommand = React.lazy(() => import('../pages/HomeRecommand'))
const HomeRanking = React.lazy(() => import('../pages/HomeRanking'))

const routes = [
  {
    path: '/',
    element: <Navigate to='/home'/>
  },
  {
    path: '/home',
    element: <Home />,
    children: [
      {
        path: '/home/recommand',
        element: <HomeRecommand />
      },
      {
        path: '/home/ranking',
        element: <HomeRanking />
      }
    ]
  },
  {
    path: 'profile',
    element: <Profile />
  },
  {
    path: '*',
    element: <NotFound />
  }
]

export default routes
```

在组件中使用（组件必须是函数组件）

```jsx
import React, { PureComponent } from 'react'
import { NavLink, useRoutes } from 'react-router-dom'

import BaseStyle from './style/base'

import withRouter from './hoc/withRouter'
import routes from './router'


export function App (props) {

  const { navigator } = props.router
  return (
    <BaseStyle>
      <h1>App</h1>
      <div className="header">
      </div>
      <div className="content">
        {useRoutes(routes)}
      </div>
      <div className="footer">
        <hr />
        Footer
      </div>
    </BaseStyle>
  )
}

export default withRouter(App)
```



## 第四站

### Hook

Hook 是 React 16.8 的新增的特性，它可以让我们在不编写 class 的情况下使用 state 以及其他的 React 特性（比如生命周期）

class 组件和函数式组件的对比：

- class 组件可以定义自己的 state，用来保存组件自己内部的状态
  - 函数式组件不可以，因为函数每次调用都会产生新的临时变量
- class 组件有自己的声明周期，我们可以在对应的生命周期中完成自己的逻辑
  - 比如在 componentDidMount 中发送网络请求，，并且该声明周期函数只会执行一次
  - 函数式组件在学习 hooks 之前，如果在函数中发送网络请求，意味着每次重新渲染都会重新发送一次网络请求
- class 组件可以在状态改变时只会重新执行 render 函数以及我们希望重新调用的生命周期函数 componentDidUpdate 等
  - 函数式组件在重新渲染时，整个函数都会被执行，似乎没有什么地方可以让他们只调用一次

class 组件存在的问题：

- 我们在最初编写一个 class 组件时，往往逻辑比较简单，并不会非常复杂，但是随着业务的增多，我们的 class 组件会变得越来越复杂。比u componentDidMount 中可能会包含大量的逻辑代码，包括网络请求，一些事件的监听（还需要再 componentWillUnmount 中移除）。对于这样的 class 实际上非常难拆分，因为他们的逻辑往往混在一起，强行拆分反而会照成设计过度，增加代码的复杂度
- 再前面为了一些状态的复用，我们需要使用告诫组件
- redux 中的 connect 或者 react-router 中的 withRouter，这些高阶组件的设计的目的就是为了状态的复用
- 类似于 Provider、Consumer 来共享一些状态，但是多次使用 Consumer 时，我们的代码会存在多层嵌套，这些代码让我们不管时在编写和设计上来说，都变得非常困呐

只能在函数最外层调用 Hook，不要再循环、条件判断或者钩子函数中调用

只能在 React 的函数组件中调用 Hook，不要在其他 JavaScript 函数中调用

#### useState

useState 可以传入一个函数，该函数的返回值作为 state

```jsx
import React, { memo, useState } from 'react'

const Counter = memo(() => {
  const [count, setCount] = useState(1)
  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={e => setCount(count + 1)}>Add 1</button>
      <button onClick={e => setCount(count - 1)}>Min 1</button>
    </div>
  )
})

export default Counter
```

#### useEffect

Effect Hook 可以用来完成一些类似于 class 声明周期的功能

网络秦桧去，手动更新 DOM，一些事件的监听，都是 React 更新 DOM 的一些副作用，对于完成这些功能的 Hook 被称之为 Effect Hook

- 通过使用 useEffect，可以告诉 React 需要在渲染后执行某些操作
- useEffect 传入的回调将会在 React 执行完成更新 DOM 操作之后，被执行
- 默认情况下，无论是第一次渲染之后，还是每次更新之后，都会执行这个回调函数
- useEffect 的回调函数中可以返回一个回调函数，这个回调函数将会在useEffect 下一次回调函数执行前执行
- 一个函数组件中可以编写多个 useEffect ，这些回调函数将会依次执行

```jsx
import React, { memo, useEffect, useState } from 'react'

const ChangeTitle = memo((props) => {
  const [counter, setCounter] = useState(0)

  useEffect(() => {
    // 当前传入的回调函数会在组件被渲染完成后执行
    // 网络请求/DOM 操作/事件监听 可以在此时
    console.log('modify counter', counter)
    document.title = counter
  }, [counter]) // 当 counter 发生变化时，才会执行

  useEffect(() => {
    // 这个回调类似于 componentDidMount
    console.log('event on')

    return () => {
      // 这个回调类似于 componentDidUnmount
      console.log('cancel event')
    }
  }, []) // 当组件挂载时才会执行
  
  return (
    <div>
      <h2>Count: {counter}</h2>
      <button onClick={e => setCounter(counter + 1)}>Change Counter</button>
    </div>
  )
})

export default ChangeTitle
```

#### useContext

当 Provider 提供的数据发生变化时，使用该数据对应的组件就会重新渲染

创建上下文对象

```js
import { createContext } from 'react'

const UserContext = createContext()
const ThemeContext = createContext()

export {
  UserContext,
  ThemeContext
}
```

在组件中使用 Provider

```jsx
import React from 'react'
import { createRoot } from 'react-dom/client'

import App from './App'
import { ThemeContext, UserContext } from './context'

const root = createRoot(document.querySelector('#root'))
root.render(
  <ThemeContext.Provider value={{color: 'red', fontSize: '20px'}}>
    <UserContext.Provider value={{username: 'zhangsan', age: 19}}>
      <App />
    </UserContext.Provider>
  </ThemeContext.Provider>
)
```

在子组件中使用 useContext

```jsx
import React, { memo, useContext } from 'react'
import { ThemeContext, UserContext } from '../context'

const UseContext = memo((props) => {
  const user = useContext(UserContext)
  const theme = useContext(ThemeContext)
  
  return (
    <div>
      <h2 style={{color: theme.color, fontSize: theme.fontSize}}>UseContext</h2>
      <p>{user.username} - {user.age}</p>
    </div>
  )
})

export default UseContext
```

#### useReducer

useReducer 仅仅是 useState 的一种替代方案

- 在某些场景下，如果 state 的处理逻辑比较复杂，我们可以通过 useReducer 来对其进行拆分
- 或者这些修改的 state 需要依赖之前的 state 时，也可以使用

```jsx
import React, { memo, useReducer } from 'react'


function reducer(state, action) {
  switch(action.type) {
    case 'increase':
      return { ...state, count: state.count + 1 }
    case 'decrease':
      return { ...state, count: state.count - 1 }
    case 'add_user':
      return { ...state, users: action.payload }
    default:
      return state
  }
}

const UseReduce = memo(() => {
  const [state, dispatch] = useReducer(reducer, {count: 0, users: [], recommands: []})

  return (
    <div>
      <h2>UseReducer: {state.count}</h2>
      <button onClick={e => dispatch({type: 'increase'})}>+</button>
      <button onClick={e => dispatch({type: 'decrease'})}>-</button>
      {
        state.users.map(user => (
          <li key={user.id}>{user.username} - {user.age}</li>
        ))
      }
      <button onClick={e => dispatch({type: 'add_user', payload: [{id: 1, username: 'zhangsan', age: 19}]})}>Add User</button>
    </div>
  )
})

export default UseReduce
```

#### useCallback

useCallback 的实际目的是为了进行性能的优化

- useCallback 会返回一个函数的 memoized （记忆的值）
- 在依赖不变的情况下，动词定义的时候，返回的值是相同的
- 使用 useCallback 的目的是不希望子组件进行多次渲染，而不是为了缓存函数

```jsx
import React, { memo, useCallback, useRef, useState } from 'react'


const Child = memo((props) => {
  const {increase} = props

  console.log('child component update')
  return (
    <div>
      <h2>Child Component</h2>
      <button onClick={increase}>Child Increase</button>
    </div>
  )
})

const UseCallback = memo(() => {
  const [count, setCounter] = useState(0)
  const [message, setMessage] = useState('hello')

  // 在组件每次更新时，都会生成一个新的 increase 函数，将会触发 Child 组件更新
  // function increase() {
  //   setCounter(count + 1)
  // }

  // 使用 useCallback 将会记忆之前的 回调 函数，当 count 发生变化时，才会使用新的回调
  // const increase = useCallback(function () {
  //   setCounter(count + 1)
  // }, [count]) // 当不指定 依赖的对象时，会产生闭包陷阱，无论执行多少次 increase 函数，count 仍然为原来的值 0，界面不会发生更新

  // 进一步进行优化，当修改 count 时，该函数将不更新
  const countRef = useRef(count) // 使用 useRef 将会创建一个不变对象，该对象永远指向设置的值
  countRef.current = count
  const increase = useCallback(function() {
    setCounter(countRef.current + 1)
  }, [])

  return (
    <div>
      <h2>UseCallback: {count}</h2>
      <button onClick={increase}>Increase</button>
      <button onClick={e => setMessage(Math.random())}>changeMessage - {message}</button>
      <Child increase={increase}></Child>
    </div>
  )
})

export default UseCallback
```

#### useMemo

useMemo 和 useCallback 都是对传入的东西进行缓存（类似于单例），useMemo 执行的结果是传入回调的返回值，useCallback 执行的结果是传入的回调函数

- 进行大量计算操作，需要让相关函数在每次渲染后都不重新执行
- 对于子组件传递相同对象时，使用 useMemo 进行性能优化

```jsx
import React, { memo, useMemo, useRef, useState } from 'react'

function calTotal(num) {
  console.log('execute cal total, only be executed once')
  for (let i = 0; i < 100; ++ i) num = num + i
  return num
}

const Child = memo((props) => {
  const user = props.user
  console.log('child component update only once')
  return (
    <div>
      <h2>Child - {user.username} - {user.age}</h2>
    </div>
  )
})

const UseMemo = memo(() => {
  const [count, setCount] = useState(0)
  const total = useMemo(() => {
    // 通过 useMemo 缓存计算的结果，该回调只会被执行一次
    return calTotal(40)
  }, [])

  const countRef = useRef(count)
  countRef.current = count
  // 通过 useMemo 模仿 useCallback
  const increase = useMemo(() => {
    return () => {
      setCount(countRef.current + 1)
    }
  }, [])

  // 此时 user 的内存地址将不会发生改变，子组件将不会重新渲染
  const user = useMemo(() => ({username: 'zhangsan', age: 19}), [])

  return (
    <div>
      <h2>UseMemo: count: {count} - total: {total}</h2>
      <button onClick={increase}>Increase</button>
      <Child user={user}/>
    </div>
  )
})

export default UseMemo
```

#### useRef

useRef 返回一个 ref 对象，返回的 ref 对象在组件的整个声明周期保持不变

```jsx
import React, { memo, useCallback, useRef, useState } from 'react'

// 通过 obj 也可以解决闭包陷阱
const obj = {count: 0}

const UseRef = memo(() => {
  const [count, setCount] = useState(0)
  const elementRef = useRef()

  function getRef() {
    console.log(elementRef.current)
  }
  // 通过定义全局对象可以达到和 useRef 同样的效果
  // obj.count = count
  // const increase = useCallback(() => {
  //   // setCount(count + 1)
  //   setCount(obj.count + 1)
  // }, [])
  
  const countRef = useRef(count)
  countRef.current = count
  const increase = useCallback(() => {
    setCount(countRef.current + 1)
  }, [])

  return (
    <div>
      <h2 ref={elementRef}>UseRef: {count}</h2>
      <button onClick={getRef}>Get Ref</button>
      <button onClick={increase}>Increase</button>
    </div>
  )
})

export default UseRef
```

#### useImperativeHandler

```jsx
import React, { forwardRef, memo, useImperativeHandle, useRef } from 'react'

const Child = memo(forwardRef((props, ref) => {
  const inputRef = useRef()

  // 通过 useImperativeHandler 可以限制父组件对子组件的控制权限
  ref = useImperativeHandle(ref, () => {
    return {
      focus() {
        inputRef.current.focus()
      }
    }
  })
  return (
    <div>
      <h2>Child</h2>
      <input type="text" ref={inputRef} />
    </div>
  )
}))

const UseImperativeHandle = memo(() => {
  const inputRef = useRef()
  function focus(){
    inputRef.current.focus()
    console.log(inputRef.current.value)
  }
  return (
    <div>
      <h2>UseImperativeHandle</h2>
      <Child ref={inputRef}></Child>
      <button onClick={focus}>Focus</button>
    </div>
  )
})

export default UseImperativeHandle
```

#### useLayoutEffect

- useEffect 会在渲染的内容更新到 DOM 上后执行，不会阻塞 DOM 的更新
- useLayoutEffect 会在渲染的内容更新到 DOM 上之前执行，会阻塞 DOM 的更新

官方更推荐使用 useEffect 而不是 useLayoutEffect

```jsx
import React, { memo, useEffect, useLayoutEffect } from 'react'

const UseLayoutEffect = memo(() => {
  useLayoutEffect(() => {
    console.log('useLayoutEffect')
  })
  useEffect(() => {
    console.log('useEffect')
  })
  return (
    <div>UseLayoutEffect</div>
  )
})

export default UseLayoutEffect
```

#### useTransition

返回一个状态值表示过度任务的等待状态，以及启动一个该过度任务的函数。它告诉 react 对于某部分任务的更新优先级较低，可以稍后进行更新

```jsx
import React, { memo, useState, useTransition } from 'react'

const itemList= new Array(1000).fill(1).map((item, index) => 'index' + index)

const App = memo((props) => {
  const [items, setItems] = useState(itemList)
  // pending 代表当前状态，如果是在执行 setTransition 将会是 true，否则 false
  const [pending, setTransition] = useTransition()

  function valueChangeHandle(e) {
    // 此时由于界面发生较大改动，会导致渲染过程很慢，页面会卡顿
    // setItems(itemList.filter(item => item.includes(e.target.value)))

    setTransition(() => {
      setItems(itemList.filter(item => item.includes(e.target.value)))
    })
  }

  return (
    <div>
      <h1>App</h1>
      <input type="text" onInput={valueChangeHandle} />
      { pending && <p>is ready to loading</p>}
      <ul>
        {
          items.map(item => {
            return <li key={item}>{item}</li>
          })
        }
      </ul>
    </div>
  )
})


export default App
```

#### useDeferredValue

useDeferredValue 接受一个值，并返回该值的副本，该副本将推迟到更紧急的更新之后。他和 useTransition 类似

```jsx
import React, { memo, useDeferredValue, useState, useTransition } from 'react'

const itemList= new Array(1000).fill(1).map((item, index) => 'index' + index)

const App = memo((props) => {
  const [items, setItems] = useState(itemList)
  // 将返回 items 的副本
  const deferredItems = useDeferredValue(items)
  // pending 代表当前状态，如果是在执行 setTransition 将会是 true，否则 false
  const [pending, setTransition] = useTransition()

  function valueChangeHandle(e) {
    // 此时由于界面发生较大改动，会导致渲染过程很慢，页面会卡顿
    // setItems(itemList.filter(item => item.includes(e.target.value)))

    setTransition(() => {
      setItems(itemList.filter(item => item.includes(e.target.value)))
    })
  }

  return (
    <div>
      <h1>App</h1>
      <input type="text" onInput={valueChangeHandle} />
      { pending && <p>is ready to loading</p>}
      <ul>
        {
          // 此时的界面渲染将会被推迟
          deferredItems.map(item => {
            return <li key={item}>{item}</li>
          })
        }
      </ul>
    </div>
  )
})


export default App
```





### 自定义 Hook

#### 自定义生命周期 Hook

```jsx
import React, { memo, useEffect, useState } from 'react'

/*
 输出顺序：
 child componentDidMount
 father componentDidMount
 child componetWillUnmount （点击 toggle 按钮后）
 child componentDidMount （再次点击 toggle 按钮后）
*/
const lifeStyle = (cname) => {
  useEffect(() => {
    console.log(cname, 'componentDidMount')
    return () => {
      console.log(cname, 'componetWillUnmount')
    }
  }, [])
}

const Child = memo(() => {
  lifeStyle('child')
  return (
    <h2>Child</h2>
  )
})

const CustomerLifeStyle = memo(() => {
  const [isShow, setIsShow] = useState(true)
  lifeStyle('father')
  return (
    <div>
      <h2>LifeStyle</h2>
      <button onClick={e => setIsShow(!isShow)}>toggle destroy</button>
      { isShow && <Child />}
    </div>
  )
})

export default CustomerLifeStyle
```

#### 自定义监控滚动位置 Hook

```jsx
import { useEffect, useState } from "react"

const useScroll = () => {
  const [scrollX, setScrollX] = useState(0)
  const [scrollY, setScrollY] = useState(0)

  useEffect(() => {
    window.addEventListener('scroll', (e) => {
      setScrollX(window.scrollX)
      setScrollY(window.scrollY)
    })

    return () => {
      window.removeEventListener('scroll')
    }
  }, [])

  return [scrollX, scrollY]
}

import React, { memo } from 'react'

const CustomerScroll = memo(() => {
  const [scrollX, scrollY] = useScroll()
  return (
    <div style={{height: '50vh', width: '150vh'}}>CustomerScroll - [{scrollX}] - [{scrollY}]</div>
  )
})

export default CustomerScroll
```

### Redux 中的 Hook

useSelector 用于获取当前的 state

useDispatch 用于分发事件

定义 store

```js
import { configureStore } from '@reduxjs/toolkit'
import counterReducer from './features/count'

const store = configureStore({
  reducer: {
    counter: counterReducer
  }
})

export default store
```

定义 reducer

```js
import { createSlice } from "@reduxjs/toolkit"

const counterSlice = createSlice({
  name: 'counter',
  initialState: {
    count: 10
  },
  reducers: {
    addCount(state, action) {
      state.count += action.payload
    },
    minCount(state, action) {
      state.count -= action.payload
    }
  }
})

export const { addCount, minCount } = counterSlice.actions

export default counterSlice.reducer
```

在组件中使用

```jsx
import React, { memo } from 'react'
import { connect, useDispatch, useSelector } from 'react-redux'
import { addCount, minCount } from './store/features/count'

const App = memo((props) => {
  const { count, addCounter, minCounter } = props

  // 使用 redux 钩子函数引入
  const state = useSelector((state) => {
    return state.counter
  })
  const dispatch = useDispatch()
  
  return (
    <div>
      <h1>App</h1>
      <p>{count}</p>
      <button onClick={e => addCounter(1)}>Add</button>
      <button onClick={e => minCounter(1)}>Min</button>

      <hr />

      <h2>New App</h2>
      <p>{state.count}</p>
      <button onClick={e => dispatch(addCount(1))}>Add</button>
      <button onClick={e => dispatch(minCount(1))}>Min</button>
    </div>
  )
})

// 之前引入 redux 方式
const mapStateToProps = (state) => ({
  count: state.counter.count
})
const mapDispatchToProps = (dispatch) => ({
  addCounter: (num) => dispatch(addCount(num)),
  minCounter: (num) => dispatch(minCount(num))
})

export default connect(mapStateToProps, mapDispatchToProps)(App)
```



当使用 useSelector 获取状态时，当改变状态时，所有使用到 useSelector 的组件都会重新渲染，使用 shallowEqual 进行浅层比较

```jsx
import React, { memo } from 'react'
import { shallowEqual, useDispatch, useSelector } from 'react-redux'
import { addCount, minCount, changeMessage } from './store/features/count'

const Home = memo(() => {
  console.log('Home render')
  // 此时，父组件更改 state ，该组件将会被重新渲染
  // const { message } = useSelector((state) => state.counter)
  // 使用 shallowEqual 父组件更改的内容子组件没有使用，子组件将不会重新渲染
  const message = useSelector((state) => state.counter.message, shallowEqual)
  const dispatch = useDispatch()

  return (
    <div>
      <h2>Home: {message}</h2>
      <button onClick={e => dispatch(changeMessage('home'))}>Change Message</button>
    </div>
  )
})

const App = memo((props) => {
  // 使用 redux 钩子函数引入
  const state = useSelector((state) => {
    return state.counter
  })
  const dispatch = useDispatch()
  console.log('App render')
  return (
    <div>
      <h1>App</h1>
      <p>{state.count}</p>
      <button onClick={e => dispatch(addCount(1))}>Add</button>
      <button onClick={e => dispatch(minCount(1))}>Min</button>
      <Home />
    </div>
  )
})


export default App
```



### 服务端渲染

#### useId

useId 是一个用于生成横跨服务器端和客户端的稳定的唯一 ID 的同时避免 hydration 不匹配的 Hook

- useId 是用于 react 的同构应用开发的，前端的 SPA 页面并不需要使用
- useId 可以保证应用程序在客户端和服务端生成唯一的 ID，这样可以有效的避免通过一些手段生成的 id 不一致，造成 hydration mismatch



### 项目搭建

由于使用 create-react-app 命令创建项目时，所有关于 webpack 的配置都被隐藏可以通过如下方式进行项目的配置

1. 使用 eject 命令导出（不可逆）`npm run eject` 
2. 使用 craco `npm install @craco/craco@alpha -D` （推荐）

create-react-app config

craco 进行对项目进行配置 [详细配置信息](https://craco.js.org/docs/configuration/devserver/) 

```js
const path = require('path')

module.exports = {
  webpack: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@utils': path.resolve(__dirname, './src/utils')
    },
    configure: {
      resolve: {
        extensions: ['.jsx', 'js', '.ts', '.tsx', '.css', '.scss', '.less']
      }
    }
  },
  devServer: {
    host: '0.0.0.0',
    port: 9000,
    open: false
  }
}
```

并需要在 package.json 文件中使用 craco 命令启动项目

```json
  "scripts": {
    "start": "craco start",
    "build": "crac0 build",
    "test": "craco test",
    "eject": "react-scripts eject"
  }
```

