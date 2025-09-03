---
title: JS 高级内容
date: 2025-04-04 19:31:42
tags: [JS]
categories: [编程]
---

## 基础

基本数据类型：number、string、undefined、null、boolean、symbol

引用数据类型：Array、Object、Function、Date、Regex



## 块级作用域

let 和 const

- 全局定义的变量不再作为属性添加到全局对象中
- 在变量定义之前使用它会报错
- 不可重复定义同名变量
- 使用 const 定义变量时，必须初始化
- 变量具有会计作用域，在代码块之外不可以使用
  - 在 for 循环中使用 let 定义变量，变量所在的作用域是循环体，也因此在循环外不能使用。
  - 另外 for 循环会对该变量做特殊处理，让每次循环使用的都是一个独立的循环变量，这可以解决 JS 由来已久的问题



## class 和 function ：

**相同点：**

1.  class 和 function 都可以作为构造函数，通过 new 操作符来实例化。 
2.  类可以包含构造函数方法、实例方法、setter 函数、getter 函数和静态类方法，但这些 **都不是** 必须的。 

**不同点：**

1. class 构造函数必须使用 new 操作符

2. class 声明不可以提升

   ```js
   // file 1
   const person1 = new Person('person1')
   console.log(person1) // Person { name: 'person1' }
   
   function Person(name) {
     this.name = name
   }
   
   // file 2
   const person2 = new Person('person2') // ReferenceError: Cannot access 'Person' before initialization
   
   class Person {
     constructor(name) {
       this.name = name
     }
   }
   ```

3. class 不可以用 call、apply、bind 改变执行上下文



**node 事件循环和浏览器事件循环区别** 

**组合式 和 声明式 api 他们之间冲突，会优先使用哪个？** 

如何将响应式数据变为非响应式



## 单例模式

```js
function signle() {
  let obj

  function _signle(o) {
    if (obj) {
      return obj
    }
    obj = o
    return obj
  }

  return _signle
}

let obj = { name: 'lili', age: 18 }
let signleObj = signle()
let a = signleObj(obj)
let b = signleObj(obj)

console.log(a == b) // true
console.log('a: ', a) // a:  { name: 'lili', age: 18 }
console.log('b: ', b) // b:  { name: 'lili', age: 18 }
```



## 手写 call、apply、bind

- `call` 是一个一个地传递参数
- `apply` 是通过数组来传递参数
- `bind` 方法在创建新函数时可以预先传递部分参数，后续调用新函数时再传递剩余参数。

```js
let obj = { name: 'lili', fn: () => 'hello' }

test.call(obj, 2, 3, 4) // { name: 'lili', fn: [Function: fn] } [ 2, 3, 4 ]
test.apply(obj, [2, 3, 4]) // { name: 'lili', fn: [Function: fn] } [ 2, 3, 4 ]
const bindTest = test.bind(obj, 2)
bindTest(3, 4) // { name: 'lili', fn: [Function: fn] } [ 2, 3, 4 ]
```



**test 函数 ** 

```js
function test(...args) {
  console.log(this, args)
}
```



**手写 call** 

```js
Function.prototype.myCall = function (obj, ...args) {
  obj = obj == null || obj == undefined? globalThis : Object(obj)

  let key = Symbol('key')
  
  obj[key] = this
  const result = obj[key](...args)
  
  delete obj[key]

  return result
}

function test(...args) {
  console.log(this, args)
}

let obj = { name: 'lili', fn: () => 'hello' }
test.call(obj, 1, 2, 3) // { name: 'lili', fn: [Function: fn] } [ 1, 2, 3 ]
test.myCall(obj, 1, 2, 3) // { name: 'lili', fn: [Function: fn], [Symbol(key)]: [Function: test] } [ 1, 2, 3 ]
```

**手写 bind 函数** 

```js

let obj = { name: 'lili', fn: () => 'hello' }


Function.prototype.myBind = function (obj, ...args) {
  let that = this
  
  return function (...rest) {
    const key = Symbol('fn')
    obj[key] = that
    // Object.defineProperty(obj, key, that)
    let result
    console.log(new.target)
    if (new.target) { // 是否通过 new 关键字调用，当通过 new 调用时：[Function (anonymous)]， 当之间调用时：undefined
      result = new obj[key](...args, ...rest)
    } else {
      result = obj[key](...args, ...rest)
    }
    delete obj[key]
    return result
  }
}

const testBind = test.myBind(obj, 2, 3)
testBind(4, 5) // { name: 'lili', fn: [Function: fn], [Symbol(fn)]: [Function: test] } [ 2, 3, 4, 5 ]
```





## global 和 window

- **使用环境不同**：`window` 主要用于浏览器环境，而 `global` 主要用于 Node.js 环境。这是它们最本质的区别，因为它们分别对应了不同的 JavaScript 运行平台。
- **功能内容不同：** 
  - `window` 包含了许多与浏览器操作相关的属性和方法，如 DOM 操作、浏览器窗口控制、定时器等，这些功能是为了实现网页的展示和交互。
  - `global` 主要围绕 Node.js 的模块系统和服务器端运行相关的一些功能，如模块加载等。虽然它也作为全局变量和函数的挂载点，但具体的功能实现和 `window` 有很大差异。
- **对象属性差异**：在浏览器环境中，`window` 对象具有非常丰富的属性，其中一些属性在 Node.js 的 `global` 对象中是不存在的，反之亦然。
  - `window` 有 `location` 属性用于获取和设置浏览器的 URL 相关信息而 `global` 在 Node.js 环境中没有这个属性；
  - `global` 中的 `require` 属性用于模块加载，在浏览器环境的 `window` 对象中通常没有（除非通过一些特殊的构建工具或者浏览器扩展等方式引入类似功能）。

## ES Modules、CommonJS

- ES Modules 模块加载机制

  - **静态分析**：ES Modules 的一个重要特性是在编译阶段就进行模块依赖关系的静态分析。这意味着在代码执行之前，JavaScript 引擎就能确定模块之间的依赖关系。例如，在解析`import`语句时，会根据模块路径找到对应的模块文件，并分析其中的导出内容。这种静态分析有助于进行一些优化，比如在打包工具（如 Webpack）中，可以提前确定模块的依赖树，进行代码的合并、压缩等操作，减少浏览器加载时的请求数量。
  - **异步加载特性**：在浏览器环境中，ES Modules 天然支持异步加载。当浏览器遇到`import`语句时，它会异步地请求模块文件，不会阻塞后续代码的执行（只要后续代码不依赖于这个还未加载完成的模块）。这与浏览器的异步加载资源的特性相匹配，能够更好地利用浏览器的多线程加载能力，提高网页的加载性能。在 Node.js 环境中，虽然底层实现上也是异步加载，但在实际使用中，可能需要一些配置或者工具支持来更好地发挥其异步加载的优势。

- 作用域和模块独立性

  - **严格的模块边界**：每个 ES 模块都有自己独立的作用域，模块内部定义的变量、函数等不会自动暴露到全局环境中。这有效地避免了全局变量的污染，使得代码的模块化更加清晰。例如，在一个模块中定义的变量`let privateVariable = 'This is private';`在其他模块中无法直接访问，除非通过该模块的导出机制。
  - **导入绑定的特性**：当从一个模块导入内容到另一个模块时，导入的变量实际上是一种绑定关系。例如，如果一个模块导出了一个变量`export let counter = 0;`，另一个模块导入了这个变量`import { counter } from './counterModule.js';`，当在第一个模块中修改`counter`的值时，在第二个模块中这个变量的值也会相应改变。这种绑定是基于引用的，并且是只读的（不能在导入模块中直接重新赋值给导入的变量，如`counter = 1;`会报错，但可以通过调用第一个模块提供的修改函数来间接改变变量的值）。

- CommonJS 模块加载机制

  - **同步加载**：CommonJS 模块加载是同步的。当执行`require`函数时，JavaScript 执行会暂停，等待被请求的模块加载完成并返回其`module.exports`的内容。这种同步加载方式在服务器端环境（如 Node.js）是比较合理的，因为在服务器启动阶段，模块的加载顺序和完整性很重要，而且服务器环境通常不会有像浏览器那样对加载时间非常敏感的用户体验问题。但是在浏览器环境中，同步加载模块可能会导致页面的长时间阻塞，影响用户体验。

  - 作用域和模块独立性
    - **模块级别的作用域**：和 ES Modules 类似，CommonJS 模块也有自己独立的作用域。模块内部的变量和函数不会自动暴露到全局环境中，这有助于保持代码的模块化。例如，在一个 CommonJS 模块内部定义的变量`var internalVariable = 'This is internal';`不会在其他模块中直接可见。
    - **值传递特性**：当一个模块通过`require`导入另一个模块的内容时，实际上是对`module.exports`内容的复制（对于基本类型是值复制，对于对象类型是引用复制）。例如，如果一个模块导出了一个对象`module.exports = { count: 0 };`，另一个模块导入这个对象后`const myObject = require('./myObjectModule.js');`，当在第一个模块中修改`myObject.count`的值时，在第二个模块中这个值也会改变（因为对象是引用类型）。但是如果在第一个模块中重新赋值`module.exports`（如`module.exports = { newCount: 1 };`），在第二个模块中`myObject`的值不会随之改变，因为这是一种值传递的关系。

简化 module.exports 实现

```js
// 用于存储已经加载过的模块，避免重复加载
const loadedModules = {}

function require(modulePath) {
  // 根据传递的模块路径，得到模块的绝对路径
  const moduleId = getModuleId(modulePath)

  // 判断是否已有缓存
  if (loadedModules[modulePath]) {
    return loadedModules[modulePath].exports
  }

  // 导入的模块将会复制到 _require 函数中，模块的执行上下文将会存在该函数的参数
  function _require(exports, require, module, __filename, __dirname) {
    // 目标模块的代码
  }

  // 创建一个模块对象，包含 exports 属性用于存储要导出的内容
  const module = { exports: {} }

  const exports = module.exports
  const __filename = moduleId
  const __dirname = getDirname(__filename)

  _require.call(exports, exports, require, module, __filename, __dirname)

  // 将模块添加到已加载模块列表中，虽然此时 exports 还为空，但避免后续重复创建模块对象
  loadedModules[modulePath] = module.exports

  return module.exports
}
```



## Promise

promise 的链式调用



- `Promise.all()`中的 Promise 序列会全部执行通过才认为是成功，否则认为是失败；
- `Promise.race()`中的 Promise 序列中第一个执行完毕的是通过，则认为成功，如果第一个执行完毕的 Promise 是拒绝，则认为失败；
- `Promise.any()`中的 Promise 序列只要有一个执行通过，则认为成功，如果全部拒绝，则认为失败；

### Promise.all

```js
Promise.myAll = function(proms) {
  let resolve, reject
  const promise = new Promise((res, rej) => { resolve = res, reject = rej })

  let len = 0, res = []
  
  for (let prom of proms) {
    const index = len;
    len ++
    Promise.resolve(prom).then((data) => {
      res[index] = data
      len --;
      if (len == 0) {
        resolve(res)
      }
    }, reject)
  }
  if (len == 0) {
    resolve([])
  }
  return promise
}

Promise.myAll([1, 2, Promise.reject(3), Promise.reject(4), 5]).then((datas) => {
  console.log(datas)
}).catch(err => {
  console.log(err) // 3
})

Promise.myAll([1, 2, Promise.resolve(3), Promise.resolve(4), 5]).then((datas) => {
  console.log(datas) // [ 1, 2, 3, 4, 5 ]
}).catch(err => {
  console.log(err)
})
```

### Promise.race

```js
Promise.race = function(promises) {
  return new Promise((resolve, reject) => {
    for (let promise of promises) {
      promise.then(res => {
        resolve(res)
      }).catch(err => {
        reject(err)
      })
    }
  })
}

const promise1 = new Promise(resolve => { setTimeout(() => resolve('promise1'), 1001) })
const promise2 = new Promise(resolve => { setTimeout(() => resolve('promise2'), 1000) })
Promise.race([promise1, promise2]).then(res => {
  console.log(res)
}) // promise2
```

### Promise.race

```js

Promise.any = function(promises) {
  let result = []
  let index = 0
  let count = 0
  return new Promise((resolve, reject) => {
    for (let promise of promises) {
      let i = index
      index ++
      promise.then(res => {
        resolve(res)
      }).catch(err => {
        count ++
        result[i] = err
        if (count == promises.length) {
          reject(result)
        }
      })
    }
  })
}

const p1 = new Promise((resolve, reject) => { setTimeout(() => reject('p1'), 1001) })
const p2 = new Promise((resolve, reject) => { setTimeout(() => reject('p2'), 1000) })
Promise.any([p1, p2]).then(res => {
  console.log(res)
}).catch(err => console.log('err', err))
```

并发请求

```js
fetch = (url) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(url)
    }, 1000)
  })
}

function request(urls, max_count, callback) {
  let result = []
  let index = 0, count = 0

  function _request() {
    return new Promise((resolve, reject) => {
      let i = index
      index ++
      if (i >= urls.length) return
      fetch(urls[i]).then(res => {
        result[i] = res
      }).catch(err => {
        result[i] = err
      }).finally(() => {
        count ++
        if (count == urls.length) {
          callback(result)
        }
        if (count < urls.length) {
          _request()
        }
      })
    })
  }

  for (let i = 0; i < max_count && i < urls.length; ++ i) {
    _request()
  }
}

request(['https://cn.bing.com/search?q=1', 'https://cn.bing.com/search?q=2',
          'https://cn.bing.com/search?q=3', 'https://cn.bing.com/search?q=4',
          'https://cn.bing.com/search?q=5', 'https://cn.bing.com/search?q=6', 
          'https://cn.bing.com/search?q=7', 'https://cn.bing.com/search?q=8'], 4, (res) => {
  console.log(res)
})
```





## 链式调用和延迟执行

```js
function arrange(s) {
  let queue = [() => console.log(s)]

  function _do(do_str) {
    queue.push(() => console.log(do_str))
    return this
  }
  function wait(delay) {
    queue.push(() => new Promise(resolve => {
      setTimeout(() => {
        console.log('wait')
        resolve()
      }, delay * 1000)
    }) )
    return this
  }
  function waitFirst(delay) {
    queue.unshift(() => new Promise(resolve => {
      setTimeout(() => {
        console.log('waitFirst')
        resolve()
      }, delay * 1000)
    }))
    return this
  }

  async function execute() {
    for (let task of queue) {
      await task()
    }
  }


  return {
    execute,
    do: _do,
    wait,
    waitFirst
  }
}

// arrange('William').execute()
// arrange('William').do('commit').execute()
// arrange('William').wait(5).do('commit').execute()
arrange('William').waitFirst(5).do('push').execute()
```



## 对象原型链 prototype， `__proto__` 

![1733582096968](H:\知识网络\面试\前端方向\study\imgs\原型链)

```js
function Person() {

}
Person.prototype = { name: 'lili' }

const p = new Person()
// 类本身存在 prototype，通过该类创建的对象的 __proto__ 对象指向 prototype
console.log(Person.prototype) // { name: 'lili' }
console.log(p.__proto__)      // { name: 'lili' }
console.log(p.__proto__ == Person.prototype) // true
console.log(Person.prototype.__proto__ == Object.prototype) // true

console.log(Function.prototype.__proto__ == Object.prototype) // true
console.log(Person.__proto__ == Function.prototype) // true

console.log(Function.prototype == Function.__proto__) // true
console.log(Object.prototype.__proto__) // null
console.log(Object.prototype.prototype) // undefined
```



## Object 方法

Object.assign()，Object函数，获取对象的类型 instanceof，使对象属性不可变（writeable=false，configurable=false，Object.preventExtensions()，Object.seal()，Object.freeze()）



## this 指向

| 调用方式          | 示例             | 函数中 this 指向 |
| ----------------- | ---------------- | ---------------- |
| 通过 new 调用     | new method()     | 新对象           |
| 直接调用          | method()         | 全局对象         |
| 通过对象调用      | obj.method()     | 前面的对象       |
| call, apply, bind | method.call(ctx) | 第一个参数       |



## XMLHttpRequest 和 Fetch



Reflect

线程进程共享资源

跨域解决方案

dom、bom

loader实现原理、eslint实现原理

js 调用执行栈

js 作用域

TCP/IP 协议

计算机网络、操作系统

死锁，如何解除死锁

Symbol、DefineProperty、Proxy，Object.property.hasOwnProperty()

浏览器加载HTML过程（重排、重绘）HTML+CSS + Javascript （JS是否会阻塞渲染队列）







script标签的属性

less、scss、css， 以及css-loader

use strict

vuex module, pinia

设计模式

MVVM、MVC

js内存回收机制

http 三次握手四次挥手

websocket

性能优化

webpack、vite（esbuild、rollup）

setup 和 setup函数

vue, react 声明周期

响应式原理、双向绑定（this.nextTick(), this.$set(), Object.defineProperty()）

render 函数和 diff 算法



## apply & call 函数

```js
Function.prototype.testCall = function(_this, ...args) {
  console.log(this, args)
  _this.fn = this
  _this.fn(args)
}
```





## 深拷贝



```js

function deepCopy(data) {
  if (typeof data === 'object' && data !== null) {
    let res = Array.isArray(data) ? [] : {}

    for (let key in data) {
      if (data.hasOwnProperty(key)) {
        res[key] = deepCopy(data[key])
      }
    }

    return res
  } else {
    return data
  }
}
```

测试：

```js

let a = {
  age: 18,
  name: 'jock',
  info: {
    addr: "abc"
  },
  func: function () {
    console.log(this.name)
  }
}
let b = deepCopy(a)

a.func = function () {
  console.log(this.age)
}
a.age = 19

console.log(b)
// {
//    age: 18,
//    name: 'jock',
//    info: { addr: 'abc' },
//    func: [Function: func]
// } 
a.func() // 19
b.func() // jock
```





## 闭包

JavaScript 闭包的设计思想来源于 Scheme

- 一个函数和对其周围状态（lexical environment，词法环境）的引用捆绑在一起（或者说函数被引用包围），这样的组合就是闭包
- 闭包可以让你在一个内层函数中访问到外层函数的作用域
- 在 JavaScript 中，每当创建一个函数，闭包就会在函数创建的同时被创建出来

## 防抖

当使用按钮点击或者输入框事件监听时，``onclick`` 或 ``oninput`` 函数将会被频繁触发，为了解决这一问题：

```js
export function debounce(callback, delay=1000, immidate=false) {
  let timer = null;

  function _debounce(...args) {
    if (timer != null) clearTimeout(timer)
    
    if (immidate === true) {
      callback.apply(this, args)
      immidate = false;
    }

    timer = setTimeout(() => {
      callback.apply(this, args)
      // callback(args)
    }, delay)
  }

  return _debounce
}
```

## 节流

在固定的时间段内只会执行一次，不管触发了多少次。例如子弹射击游戏中，射击按键可能会快，执行射击不会一直往后延迟，而会在固定的频率下执行射击动作

```js
export function throttle(callback, interval, immidate = true) {
  let start_time = 0  

  function _throttle(...args) {
    let now_time = new Date().getTime()

    if (immidate === false && start_time === 0) {
      start_time = now_time
    }

    let rest_time = interval - (now_time - start_time)
    
    if (rest_time <= 0) {
      callback.apply(this, args)
      start_time = now_time
    }
  }

  return _throttle
}
```

尾部执行

```js
export function throttle(callback, interval, immidate = true, trailing = false) {
  let start_time = 0  
  let timer = null

  function _throttle(...args) {
    let now_time = new Date().getTime()

    if (immidate === false && start_time === 0) {
      start_time = now_time
    }

    let rest_time = interval - (now_time - start_time)

    if (rest_time <= 0) {
      clearTimeout(timer)
      timer = null
      callback.apply(this, args)
      start_time = now_time
    }

    if (trailing && timer === null) {
      timer = setTimeout(() => {
        timer = null
        start_time = new Date().getTime()
        callback.apply(this, args)
      }, rest_time)
    }
  }

  return _throttle
}
```



## 函数重载

```js
function addMethod(obj, name, fn) {
  const old = obj[name]
  obj[name] = function(...args) {
    if (args.length == fn.length) {
      return fn.apply(this, args)
    } else if (typeof old === 'function') {
      return old.apply(this, args)
    }
  }
}
```

测试代码

```js
let obj = {}
addMethod(obj, 'getUser', () => {
  console.log('no argument function')
})

addMethod(obj, 'getUser', (a) => {
  console.log('one argument function, argument: ', a)
})

addMethod(obj, 'getUser', (a, b) => {
  console.log('two arguments function, argument: ', a, b)
})

obj.getUser() // no argument function
obj.getUser('1') // one argument function, argument:  1
obj.getUser('1', '2') // two arguments function, argument:  1 2
```

如上代码存在一些问题：

- 当存在默认参数时，会失效

  ```js
  let obj = {}
  addMethod(obj, 'getUser', () => {
    console.log('no argument function')
  })
  
  addMethod(obj, 'getUser', (a = 1) => {
    console.log('one argument function, argument: ', a)
  })
  
  addMethod(obj, 'getUser', (a, b) => {
    console.log('two arguments function, argument: ', a, b)
  })
  
  obj.getUser() // one argument function, argument:  1
  obj.getUser('1') //
  obj.getUser('1', '2') // two arguments function, argument:  1 2
  ```

- 当同为一个参数，但参数类型不一致也会失效

改进后的函数重载方法

```js
function createOverload() {
  const map = new Map()

  function overload(...args) {
    const key = args.map(arg => typeof arg).join(',')
    const fn = map.get(key)
    if (!fn) {
      throw new TypeError('no implement function')
    }
    fn.apply(this, args)
  }

  overload.addImpl = function (...args) {
    let fn = args.pop()
    if (typeof fn !== 'function') {
      throw new TypeError('last argument must be function')
    }
    const key = args.join(',')
    map.set(key, fn)
  }

  return overload
}
```

测试代码

```js
const fun = createOverload()

fun.addImpl(() => {
  console.log('no argument function')
})

fun.addImpl('number', (...args) => {
  console.log('one number argument function')
})

fun.addImpl('string', (...args) => {
  console.log('one string argument function')
})

fun.addImpl('string', 'string', (...args) => {
  console.log('two string arguments function')
})

fun() // no argument function
fun(123) // one number argument function
fun('str') // one string argument function
fun('str1', 'str2') // two string arguments function
```

## 给 fetch 添加超时时间

```js
function createRequestWithTimeout(timeout = 3000) {
  return function(url, options) {
    return new Promise((resolve, reject) => {
      const abort = new AbortController()
      options = options || {}

      if (options.signal) {
        options.signal.addEventListener('abort', () => {
          abort.abort()
        })
      } else {
        options.signal = abort.signal
      }
      setTimeout(() => {
        reject(new Error('timeout'))
        abort.abort()
      }, timeout)
      
      fetch(url, options).then(resolve, reject)
    })
  }
}
```

测试代码

```js
const fetch = (url, options) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(url)
      console.log(url)
    }, 1000)
  })
}

const request = createRequestWithTimeout()
request('123')
```



## 大整数相加

```js
/**
 * 
 * @param {String} a 
 * @param {String} b 
 * @returns {String}
 */
function sum(a, b) {
  let len = Math.max(a.length, b.length)
  a = a.padStart(len, '0')
  b = b.padStart(len, '0')
  let carry = 0
  let result = ''

  for (let i = len - 1; i >= 0; -- i) {
    let sum = parseInt(a[i]) + parseInt(b[i]) + carry
    if (sum > 10) {
      carry = 1
      sum = sum - 10
    } else {
      carry = 0
    }
    result = sum + result
  }
  if (carry > 0) {
    result = '1' + result
  }

  return result
}
```

测试代码

```js
let res = sum('1234567891011121314', '3333333333333333333333333333')
console.log(res) // 3333333334567901224344454647

res = sum('123', '999')
console.log(res) // 1122
```



## 任务队列的中断和恢复

```js
/**
 * 依次顺序执行一系列任务
 * 所有任务全部完成后可以得到每个任务的执行结果
 * 需要返回两个方法，start 用于启动任务，pause 用于暂停任务
 * 每个人物具有原子性，即不可终端，只能在两个任务之间中断
 * @param  {...any} tasks 任务列表，每个任务无参，异步
 */
function processTasks(...tasks) {
  let isRunning = false
  let i = 0
  let result = []

  return {
    async start() {
      return new Promise(async (resolve) => {
        if (isRunning) {
          return
        }
        isRunning = true
        while (i < tasks.length) {
          console.log(`the ${i} task is running`)
          let res = await tasks[i]()
          console.log(`the ${i} task is finished`)
          result.push(res)
          i ++
          if (isRunning == false) {
            return
          }
        }
        isRunning = false
        resolve(result)
      })
    },
    pause() {
      isRunning = false
    }
  }
}
```

## 消除异步任务的传递性：

```js
function test1() {
  const res = fetch('test')
  return res
}

function test2() {
  const res = test1()
  return res
}

function test3() {
  const res = test2()
  return res
}

function main() {
  const res = test3()
  console.log(res)
}

function run(func) {
  const oldFetch = globalThis.fetch
  const cache = {status: 'pending', value: null}

  const newFetch = function(...args) {
    if (cache.status == 'fulfilled') {
      return cache.value
    } else if (cache.status == 'rejected') {
      return cache.value
    }
    const promise = oldFetch(...args).then(res => {
      cache.status = 'fulfilled'
      cache.value = res
    }).catch(err => {
      cache.status = 'rejected'
      cache.value = err
    })
    throw promise
  }

  this.fetch = newFetch
  try {
    func()
  } catch(e) {
    if (e instanceof Promise) {
      e.finally(() => {
        globalThis.fetch = newFetch
        func()
        globalThis.fetch = oldFetch
      })
    }
  }
  
  globalThis.fetch = oldFetch
}


globalThis.fetch = async (...args) => {
  await Promise.resolve(() => args)
  return args
}

run(main)
```

