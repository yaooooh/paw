---
title: Node 学习笔记
date: 2025-04-08 23:31:42
tags: [Node]
categories: [编程]
---



## Node.js 

使用 npm 安装第三方工具时。当父文件夹存在 node_modules 文件夹时，并执行下载命令时（在子文件夹下），将会自动下载第三方工具到父文件夹下的 node_modules 文件夹中。

### path 模块

resolve 将会从后往前解析路径，当遇到绝对路径将会停止

```js
const path = require('path')

console.log(path.join('./a.txt', './b')) // a.txt\b

console.log(path.resolve('/a.txt', './b', '/c.txt')) // D:\c.txt

console.log(path.resolve('/a.txt', '/b.txt', './c.txt')) // D:\b.txt\c.txt

console.log(path.resolve('/a.txt', './b.txt', './c.txt')) // D:\a.txt\b.txt\c.txt

console.log(path.resolve('./a.txt', './b.txt', './c.txt')) // D:\Dahui\Project\Algorithm-Exercise\u-front\npm\src\a.txt\b.txt\c.txt

console.log(path.resolve('./a.txt', './b.txt', '', './c.txt/')) // D:\Dahui\Project\Algorithm-Exercise\u-front\npm\src\a.txt\b.txt\c.txt

console.log(__dirname, __filename) // D:\Dahui\Project\Algorithm-Exercise\u-front\npm\src D:\Dahui\Project\Algorithm-Exercise\u-front\npm\src\index.js
```



## CommonJS

### ``exports`` 与 ``module.exports`` 

``exports`` 和 ``module.exports`` 等价

```js
// b.js
console.log(module.exports === exports)
```

执行：

```bash
$ node b.js
true
```

### 路径查找问题

在使用 ``require('路径')`` 时，当路径前缀不包含 ``/``、``./``、``../`` 等相对或绝对路径时

- 将会首先在 node.js 的环境中查找该模块，例如：``require('path')`` 
- 当 node.js 环境不存在该模块时，将会在该目录的 node_modules 文件夹中查找，该目录中不存在 node_modules 文件夹则会一直向上寻找该文件夹，不存在则抛出异常

当路径前缀包含 ``/``、``./``、``../`` 等相对或绝对路径时，首先会查找其路径对应的文件，当不存在该文件时：

- 当不包含文件后缀名 (``.js``、``.json``、``.node``) 时，将会自动添加后缀，进行查找
- 当添加后缀名后查找仍不存在，则会查找该路径命名的文件夹，当不存在该文件夹时抛出异常
- 当文件夹存在时，则会在文件夹中查找 ``index.js`` 文件

### 导出引用

在使用 ``require`` 和 ``module.exports`` 时，``module.exports`` 所导出的为对象的引用，在导入文件中修改导入的变量时，原变量也会发生改变

```js
// a.js
let a = {
  name: 'aaa'
}

module.exports = {
  a
}

setTimeout(() => {
  console.log('in aaa file: ', a.name)
}, 1000)

// b.js
const ma = require('./aaa')

console.log('in bbb file: ', ma)

ma.a.name = 'bbb'
```

当执行后：

```bash
$ node b.js
in bbb file:  { a: { name: 'aaa' } }
in aaa file:  bbb
```

### module 对象

当模块使用 require 引入时，该模块中的代码将会被自动执行一次。当多次引入也会只执行一次。

node.js 源码中存在一个变量 ``module.loaded`` ，初始时为 false，当进行引入时该变量将会变为 true

```js
Module {
  id: '.',
  path: '~',
  exports: {},
  filename: '~/aaa.js',
  loaded: false, // 代表是否已经加载该模块
  children: [],
  paths: [ // 当前目录及父目录下的 node_modules 文件夹，用于查找第三方工具包
    'D:\\Dahui\\Project\\Algorithm-Exercise\\node_modules',
    'D:\\Dahui\\Project\\node_modules',
    'D:\\Dahui\\node_modules',
    'D:\\node_modules'
  ]
}
```

### 加载过程

其加载过程为同步过程，当一个模块加载完成之后，才会加载另一个，此方法经常在服务端使用（由于本地加载较为迅速）。此种方式在客户端加载时会导致卡顿或阻塞。由此引申出 AMD（Asynchronous Module Definition） 和 CMD （Common Module Definition），这两种方式都采用异步加载

```js
// a.js
console.log('in aaa file')

// b.js
console.log('in bbb file')
const ma = require('./a')
console.log('in bbb file')
```

执行结果：

```bash
$ node b.js
in bbb file
in aaa file
in bbb file
```

当存在循环引入时，其执行顺序（node.js 为深度优先搜索）

- a -> b -> c
- a -> d -> c

则执行顺序为：a -> b -> c -> d



## ES-Module

在 ES6（ES2015） 时推出

### 导入导出方式

```js
// a.js
let a = "aaa"
// 定义时直接导出，该方式不能使用 as 关键字改别名
export let aa = "aaaaaa"

export {
	a
}

// b.js
// 在浏览器中导入时，必须夹后缀（.js），导入声明只能在文件顶层使用
import {a} from 'a.js'
// import * as a from 'a.js'

console.log(a)

// a.html
<script src='b.js' type="module"></script>
```

执行结果

```bash
aaa
```

当导入时的变量与该文件中的变量产生冲突时，通过 as 关键字可以在导出文件中替换名字

```js
// a.js
let a = "aaa"

export {
	a as a_a
}

// b.js
import {a_a} from 'a.js'

const a = "bbb"
console.log(a, a_a)
```

或者在导入时起别名

```js
// a.js
let a = "aaa"

export {
	a
}

// b.js
import {a as a_a} from 'a.js'

const a = "bbb"
console.log(a, a_a)
```

执行结果

```bash
bbb, aaa
```

### 导入导出优化

当管理多个导入导出文件时，可采用如下优化方式（main.js 中需要引入 utils 文件夹下的所有工具类）。创建index.js 文件导入所有工具方法，并导出，此时 main.js 中只需要引入 index.js 文件中的方法即可

```bash
-- utils
	-- parse.js
	-- time.js
	-- index.js
-- main.js
```

代码如下：

```js
// index.js
// 导出所有
export * from 'parse.js'
// 如下更清晰
expor { timeUtils } from 'time.js'

// main.js
import { parseInt, timeUtils } from 'utils/index.js'
```

### 默认导出

```js
// a.js
// 一个模块中只能存在一个默认导出
export default function() {
    console.log('default export')
}

// b.js
import aaa from 'a.js'
```

### 导入函数

默认导入只能在文件顶层，浏览器在加载 ``js`` 文件时，会直接在文件顶层扫描 import 并下载

```js
// a.js
export let a = "aaa"

// b.js
let flag = true
if (flag) {
    const importPromise = import('./a.js')
    importPromise.then((res) => {
        console.log(res.a) // aaa
    })
    
    // import('./a.js').then((res) => {
    //     console.log(res.a) // aaa
    // })
}
```

如下导入方式将会报错，因为在执行之后才能知道具体要导入的文件

```js
import {a} from 'a' + '.js'
import {a} from ('a' + '.js')
```

在 ES11（ES2020） import 中添加了如下的属性：

```json
{
    url: 'http://127.0.0.1:5500/a.js', // 加载该 js 文件所使用的 url
    resolve: ƒ
}
```

### 解析流程 TODO







## npm

当所加载的第三方工具包的入口文件不为 index.js 文件时，无法通过 ``require('第三方工具包')`` 进行导入，可在第三方工具包中创建 ``package.json`` 文件

```json
// 例如第三方工具包的入口文件为 main.js
{
    name: "第三方工具包",
    version: "1.0.0",
    main: "main.js"
    // ...
}
```

### scripts 脚本

针对于特定名称的脚本可以省略 run 参数，例如：``npm start`` ，可以省略 run 的命令如下：start、test、stop、restart

```json
{
    // ...
    scripts: {
        start: "node main.js",
        build: "webpack ..."
    }
    // ...
}
```

### 开发依赖、生产依赖

开发依赖：只在开发过程中会使用到，例如：webpack。执行命令：``npm install/i xxx --save-dev/-D`` 

生产依赖：在开发过程以及生产过程都会被使用到，例如：vue。执行命令：``npm install/i xxx --save/-S`` 

peer 依赖：在依赖本库时需要先安装该依赖的库

*全局安装命令： ``npm install xxx -g``* 

```json
{
    // ...
    dependencies: {
        "vue": "3.0"
        // ...
    },
    devDependencies: {
        "webpack": "5.0"
        // ...
    },
    peerDependencies: {
        // ...
    }
    // ...
}
```

### 版本管理

semver 版本规范 ``X.Y.Z`` 

- X 主版本号（major）：当做了不兼容的 API 修改时（可能不兼容之前的版本）
- Y 次版本号（minor）：当做了向下兼容的功能性新增（新功能增加，但兼容以前的版本）
- Z 修订号（patch）：当做了向下兼容的问题修正（没有新功能，修复了之前版本的 BUG）

``^`` 和 ``~`` 前缀

- ``x.y.z`` 代表特定的版本
- ``^x.y.z`` 代表 x 是保持不变的，y 和 z 永远安装最新的版本
- ``~x.y.z`` 表示 x 和 y 是保持不变的，z永远安装最新的版本



### 命令

```bash
# 获取缓存目录
$ npm config get cache

# 获取配置信息
$ npm config list
# ; "user" config from ~\.npmrc
#
# cache = "~/node-v18.14.0-win-x64/node_cache"
# prefix = "~/node-v18.14.0-win-x64/node_global"
# registry = "http://registry.npm.taobao.org/"
#
# ; node bin location = ~/node-v18.14.0-win-x64/node.exe
# ; node version = v18.14.0
# ; npm local prefix = ~/project
# ; npm version = 9.3.1
# ; cwd = ~/project
# ; HOME = ~
# ; Run `npm config ls -l` to show all defaults.


# 卸载某个第三方库
$ npm uninstall xxx

# 重新构建项目依赖
$ npm rebuild

# 清除缓存
$ npm cache clean

# 获取本地使用的镜像源
$ npm config get registry

# 更新镜像源
$ npm config set registry "镜像源地址"

# 登录到 npm registry
$ npm login

# 发布当前包到 npm registry
$ npm publish

# 删除发布的包
$ npm unpublish

# 让发布的包过期
$ npm deprecate
```



## npx

当使用 npx 命令执行别的第三方包命令时，将会首先在该目录下的 node_modules 文件夹下的 .bin 文件夹中查找该命令是否存在，存在则会优先执行该命令



## pnpm

每创建一个项目都需要下载对应的第三方工具包，为了解决包占用较大的问题，pnpm采用软链接、硬链接

- 扁平化：当一个工具包依赖另一个工具包时，将会直接下载到 node_modules 文件夹下，导致在``package.json`` 中没有写入依赖项就可以导入。当卸载该工具包时，其依赖包也可能被删除，由此引发导入错误
- 非扁平化：当一个工具包依赖另一个工具包时，依赖项将会下载到在该工具包中

``pnpm`` 通过设置软连接的方式将一个工具包依赖的另一个工具包保存到该工具包目录中，安装的该工具包依赖的工具包将会以硬链接的方式存储到 ``.pnpm`` 目录下



## Webpack 模块化打包工具

> Webpack 是一个静态的模块化打包工具。打包后成为最终的静态资源，用于部署到服务器中
>
> Webpack 支持 ES Module、CommonJS、AMD 等规范



对项目进行打包：

```bash
# 执行当前项目依赖的 webpack 进行打包
$ npx webpack
```

当将该命令集成到 ``package.json`` 文件中，则可以省略 ``npx`` ，``package.json`` 中的命令将会自动在该项目依赖中进行查找

```json
{
    // ...
    scripts: {
        "build": "webpack"
    }
    // ...
}
```



webpack 默认会找到 ``src/index.js`` 文件进行打包

- 可以通过参数指定要打包的文件：``npx webpack --entry ./src/main.js`` 进行修改
- 通过添加如下参数：``npx webpack --output-filename bundle.js`` 修改输出的文件名
- 通过添加如下参数：``npx webpack --output-dir build`` 修改输出的文件夹名称

### 配置文件

通过添加 webpack.config.js 文件修改 webpack 的配置信息。该文件采用 CommonJS 语法规范

```js
// webpack.config.js
const path = require('path')


module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    // path 必须指定绝对路径
    path: path.resolve(__dirname, './build')
  }
}
```

当该配置文件名字不为 webpack.config.js 时可使用 `npx webpack --config newname.config.js` 来指定

为了简化命令可以使用 `package.json` 的 scripts ：

```json
{
	//...
    scripts: {
        "build": "webpack --config newname.config.js"
    }
    //...
}
```

### 配置后缀名

```js
// webpack.config.js
const path = require('path')


module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    // path 必须指定绝对路径
    path: path.resolve(__dirname, './build')
  },
  resolve: {
    extensions: ['.js', '.jsx', '.json'] // 配置可省略的后缀名
  },
}
```



### 打包模式

可选择的打包方式为：none | development | production（默认）

| 选项        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| development | 会将 DefinePlugin 中的 process.env.NODE_ENV 的值设置为 development，为模块和 chunk 启用有效的名 |
| production  | 会将 DefinePlugin 中 process.env.NODE_ENV 的值设置为 production，为模块和 chunk 启用确定新的混淆名称，FlagDependencyUsagePlugin、FlagIncludeChunksPlugin、ModuleConcatenationPlugin、NoEmitOnErrorsPlugin 和 TerserPligin |
| none        | 不适用任何默认优化选项                                       |



```js
// webpack.config.js
const path = require('path')


module.exports = {
  mode: 'development', // none | development | production (default)
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    // path 必须指定绝对路径
    path: path.resolve(__dirname, './build')
  }
}
```



### HTML webpack plugin

将 html 文件打包生成到指定的文件夹中

安装指定插件

```bash
npm install html-webpack-plugin -D
```

使用指定插件

```js
// webpack.config.js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')


module.exports = {
  // ...
  module: {
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './index.html'
    })
  ]
}
```



### 对 React 代码进行打包

安装 react 依赖

```bash
npm install react react-dom
```

安装对应编译 jsx 代码的插件

```bash
npm install @babel/plugin-systax-jsx -D
npm install @babel/plugin-transform-react-jsx -D
npm install @babel/plugin-transform-react-display-name -D

// 或者直接安装预设

npm install @babel/preset-react -D
```

App 组件中的 jsx 代码

```jsx
import React, { memo, useState } from "react";

const Component = memo(function () {
  const [count, setCount] = useState(0)

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setCount(count + 1)}>Add</button>
    </div>
  )
})

export default Component
```

main 入口文件

```jsx
import React from 'react'
import ReactDom from 'react-dom/client'
import App from './App.jsx'

const root = ReactDom.createRoot(document.querySelector('#app'))
root.render(<App />)
```

babel.config.js

```js
module.exports = {
  presets: [
    ['@babel/preset-env', {
      corejs: 3,
      useBuiltIns: 'entry' // 表示不使用 polyfill，可选值：【false|usage(使用polyfill进行填充)|entry】
    }],
    ['@babel/preset-react'], // 解析 react 代码
  ]
}
```



### 对 TS 文件进行打包

1. 可以使用 typescript compiler 进行打包，通过 tsc file.ts 进行手动打包

2. 通过使用 ts-loader 进行整合 webpack 进行打包

   需要对应的 tsconfig.json 文件（可以通过 npx tsc -init 生成）

   会对 类型检查错误的代码进行报错

   打包的内容并不包括 polyfill 部分

   ```js
   // webpack.config.js
   const path = require('path')
   const HtmlWebpackPlugin = require('html-webpack-plugin')
   
   module.exports = {
     // ...
     resolve: {
       extensions: ['.js', '.jsx', '.json', '.ts']
     },
     module: {
       rules: [
         {
           test: /\.ts$/,
           use: ['ts-loader']
         }
       ]
     },
     plugins: [
       new HtmlWebpackPlugin({
         template: './index.html'
       })
     ]
   }
   ```

   

3. 通过 ts 预设进行打包：@babel/preset-typescript（推荐）

   但是 使用 babel-loader 将不会对 ts 代码类型检查错误进行报错

   此时会使用 polyfill 进行填充

   ```js
   // webpack.config.js
   const path = require('path')
   const HtmlWebpackPlugin = require('html-webpack-plugin')
   
   module.exports = {
     // ...
     module: {
       rules: [
         {
           test: /\.ts$/,
           use: {
             loader: 'babel-loader', // 不会对类型检查错误进行报错
             options: {
               presets: ['@babel/preset-typescript']
             },
             // loader: 'ts-loader' // 会对类型检查错误进行报错
           }
         }
       ]
     },
     plugins: [
       new HtmlWebpackPlugin({
         template: './index.html'
       })
     ]
   }
   ```

   在使用过程中既要类型检查抛出异常又要使用 polyfill 进行代码的填充，可以通过配置 package.json 脚本

   ```json
   {
     // ...
     "scripts": {
       "build": "tsc --noEmit && webpack",
       "test": "echo \"Error: no test specified\" && exit 1"
     },
     // ...
   }
   ```

   noEmit 表示不输出任何东西，也即不进行转换



### Source Map

webapck 配置文件

```js
// webpack.config.js
const path = require('path')


module.exports = {
  mode: 'production', // none | development | production (default)
  devtool: 'source-map', // reflect to the source code, will generate the filename.js.map file, if the mode is development then the devtool is source-map
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    // path 必须指定绝对路径
    path: path.resolve(__dirname, './build')
  }
}
```

编写的主文件

```js
// main.js

let name = 'abc'
console.log(name)

name = 123
console.log(name)
```

最后一行的注释指明了该打包后的文件的 source map 文件

浏览器会根据我们的注释，查找相应的 source-map，并且根据  source-map 还原我们的代码，方便进行调试

```js
(()=>{let o="abc";console.log(o),o=123,console.log(o)})();
//# sourceMappingURL=bundle.js.map
```

生成的 source-map 文件

```json
{
  "version": 3, // 当前使用的版本，也就是最新的第三版
  "file": "bundle.js", // 打包后的文件（浏览器加载的文件）
  "mappings": "MAAA,IAAIA,EAAO,MAEXC,QAAQC,IAAIF,GAEZA,EAAO,IAEPC,QAAQC,IAAIF,E", // source-map 用来和源文件映射的信息（比如位置信息等），一串 base64 VLQ（variable length quantity，可变长度值）编码
  "sources": [
    "webpack://webpack/./src/main.js"
  ], // 从哪些文件转换过来的 source-map 和打包的代码（最初始的文件）
  "sourcesContent": [
    "let name = 'abc'\r\n\r\nconsole.log(name)\r\n\r\nname = 123\r\n\r\nconsole.log(name)\r\n"
  ], // 转换前的具体代码信息（和 sources 是对应的关系）
  "names": [
    "name",
    "console",
    "log"
  ], // 转换前的变量和属性名称（因为我们目前使用的是 development 模式，所以不需要保留转换前的名称）
  "sourceRoot": "" // 所有的 sources 相对的根目录
}
```

### devtool 选项

- false：不使用 source-map，也就是没有任何和 source-map 相关的内容

- none：production 模式下的默认值（什么值都不写），不生成 source-map

- eval：development 模式下的默认值，不生成  source-map

  - 但是他会在 eval 执行的代码中，添加 // # sourceURL=;
  - 它会被浏览器在执行时解析，并且在调试面板中生成对应的一些文件目录，方便我们调试（但是此时还原的代码并不一定准确到具体的行、列）速度快

- source-map：会生成完整的 source-map 文件，一般设置在 production 模式下

- eval-source-map：会生成 sourcemap，但是 source-map 是以 DateUrl 添加到 eval 函数的后面（将 source-map 文件内容**转化为 base64 放到 eval 函数的后面**）

  ```js
  eval(... //# sourceMappingURL=data;application/json;charset=utf-8;base64,)
  ```

  

- inline-source-map：会生成 sourcemap，但是 source-map 是以 DateUrl 添加到 bundle 文件的后面（将 source-map 文件内容**转化为 base64 放到 bundle 文件的最后面**）

  ```js
  eval(... )
  //# sourceMappingURL=data;application/json;charset=utf-8;base64,
  ```

  

- cheap-source-map

  - 会生成 sourcemap，但是会更加高效一些（cheap 低开销），因为 **他没有生成列映射** （Column Mapping）
  - 因为再发开中，我们只需要行信息通常就可以定位到错误了
  - 在 development 中才会生成相应的 .js.map 文件

- cheap-module-source-map

  - 会生成 sourcemap，类似于 cheap-source-map，但是对源自 loader 的 sourcemap 处理会更好
  - 如果 loader 对我们的源代码进行了特俗的处理，比如 babel 可能会删掉空行

- hidden-source-map

  - 会生成 sourcemap，但是不会对 sourcemap 文件进行引用
  - 相当于删除了打包文件中对 sourcemap 的引用注释。如果我们手动添加进行，那么 sourcemap 就会生效了

- nosources-source-map

  - 会生成 sourcemap，但是生成的 sourcemap 只有错误信息的提示，不会生成源代码文件

### 配置插件

```js
// webpack.config.js
const path = require('path')


module.exports = {
  mode: 'production', // none | development | production (default)
  devtool: 'source-map', // reflect to the source code, will generate the filename.js.map file, if the mode is development then the devtool is source-map
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    // path 必须指定绝对路径
    path: path.resolve(__dirname, './build'),
    clean: true, // when rebuild, the output dir will be cleaned
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: {
          loader: 'babel-loader',
          options: {
            plugins: [], // add plugins
            presets: [
              '@babel/preset-env'
            ]
          }
        }
      }
    ]
  }
}
```

### 自动化编译

通过如下方式可以达到自动编译

1. webpack watch mode
2. webpack-dev-server （常用）
   - webpack-dev-server 使用了 memfs 库，不会输出任何文件，生成的中间结果在内存中
3. webpack-dev-middleware

```bash
npm install webpack-dev-server -D
```

配置 DevServer

如果配置了 devServer.static 那么该内容会覆盖默认值，也即如果这个地方没有写 public 那么 public 将不会被认为是静态资源目录

```js
// webpack.config.js
module.exports = {
  // ...
  devServer: {
    // if this is defined, then the public must be insert to this list
    static: ['public', 'temp'],
    host: '0.0.0.0',
    open: true,
    compress: true, // 进行压缩，会设置响应头中 content-encoding: gzip    
  }
}
```



### Webpack 性能优化

- 对打包结果进行优化
  - 分包处理，Vue/React 路由懒加载
  - 代码进行压缩（丑化 const message => const m）
  - 删除无用代码（tree shaking）
  - CDN 服务器（对第三方库使用 CDN）
- 对打包过程进行优化
  - 加速打包的过程（exclude/cache-loader）

#### 1. 分包处理

- 他的主要目的是将 **代码分离到不同的 bundle 中**，之后我们可以 **按需加载**，或者并行加载这些文件
- 默认情况下，所有的 JavaScript 代码（业务代码、第三方依赖、暂时没有用到的模块）在首页全部都加载，会影响首页加载的速度
- 代码可以分离出更小的 bundle，以及控制资源加载优先级，提供代码的性能。

webapck 中常用的代码分离方式：

1. 入口起点：使用 entry 配置手动分离代码

   ```js
   module.exports = {
     entry: {
       index: './src/index.js',
       main: './src/main.js'
     },
     output: {
       // 使用占位符：name 为上面的 index 和 main
       filename: '[name]-bundle.js',
       // path 必须指定绝对路径
       path: path.resolve(__dirname, './build'),
       clean: true, // when rebuild, the output dir will be cleaned
     },
   }
   ```

   当多个入口文件对同一个库进行了依赖，可以设置起共享的库

   ```js
   module.exports = {
     entry: {
       index: {
         import: './src/index.js',
         dependOn: 'shared1'
       },
       main: {
         import: './src/main.js',
         dependOn: 'shared1'
       },
       // shared 可以配置多个，将会输出到 shared-xxx 文件中，其他依赖将会对该包进行引入 
       shared1: ['react', 'react-dom']
     },
     output: {
       // 使用占位符：name 为上面的 index 和 main
       filename: '[name]-bundle.js',
       // path 必须指定绝对路径
       path: path.resolve(__dirname, './build'),
       clean: true, // when rebuild, the output dir will be cleaned
     },
   }
   ```

   

2. 防止重复：使用 Entry Dependencies 或 SplitChunksPlugin 去重和分离代码

3. 动态导入：通过模块的内联函数调用来分离代码

   通过使用 import 函数 `import('xxx.js').then(() => {{}})` 

   当使用 import 函数时，then 中可以直接拿到导入文件的 export 对象，获取导出 default 对象，可以通过 

   ```js
   import('xxx.js').then(res => {
     // 获取默认导出对象
     res.default()
     // 获取导出对象
     res.obj
   })
   ```

通过魔法注释修改打包后的文件名：

```js
// a 代表打包后的文件名，此名称替换 name 字段
import(/* webpackChunkName: 'a' */'./ts/a').then(getResult => {
  console.log(getResult('$123'))
  console.log(getResult('234'))
})
```

webpack 配置

```js
module.exports = {
  output: {
    filename: '[name]-bundle.js',
    chunkFilename: '[name]_chunk.js',
    // chunkFilename: '[id]_[name]_chunk.js', // id 为文件路径，文件名使用 _ 分割的字符串
    // path 必须指定绝对路径
    path: path.resolve(__dirname, './build'),
    clean: true, // when rebuild, the output dir will be cleaned
  },
}
```

通过配置 webpack 优化选项，设置分包方式

```js
module.exports = {
  optimization: {
    // natural: 按照数字的顺序使用 id
    // named: development 下的默认值，文件路径使用 _ 分割
    // deterministic: 确定的，在不同的编译中不变的短数字 id
    chunkIds: 'deterministic',
    splitChunks: {
      chunks: 'all', // default：async
      maxSize: 20000, // 20kb 拆分后的包最大大小，可能会大于最大值，由于一个函数或类可能很大
      minSize: 10, // 拆分后的包最小大小
      cacheGroups: { // 自定义拆包
        vendors: {
          // test: /node_modules/, // 匹配路径中包含该字符串的
          test: /[\\/]node_modules[\\/]/, // 匹配路径中包含 /node_modules/ 或者 \node_modules\
          filename: '[name]_vendors.js'
        },
        utils: { // 当编写的文件小于 minSize 时将不会被拆分
          test: /utils/,
          filename: '[name]_utils.js'
        }
      }
    }
  },
}
```

chunIds 将会指定打包后文件的名字，当设置 natural 时，当导入文件发生变化时，不利于浏览器缓存，导致重新加载

- 在开发中，推荐使用 named
- 在打包过程中，推荐使用 deterministic



### Prefetch 和 Preload

在声明 import 时，使用内置指令，告知浏览器

- prefetch（预获取）：将来某些导航下可能需要的资源
- preload（预加载）：当前导航下可能需要的资源

于 prefetch 指令相比，preload 指令由许多不同之处

- preload chunk 会在父 chunk 加载时，以并行方式开始加载，prefetch 会在父 chunk 加载结束后开始加载
- preload chunk 具有中等优先级，并立即下载。prefetch chunk 在浏览器闲置时下载
- preload chunk 会在父 chunk 中立即请求，用于当下时刻。prefetch chunk 会用于未来的某个时刻

```js
import(
  /* webpackChunkName: 'a' */
  /* webpackPreload: true */ // 设置预加载
  './ts/a').then(getResult => {
  console.log(getResult('$123'))
  console.log(getResult('234'))
})
```



### Shimming

shimming 是一个概念，是某一类功能的统称：

- shimming 翻译过来我们称之为 垫片，相当于给我们的代码填充一些垫片来处理一些问题
- 比如我们现在以来一个第三方的库，这个第三方的库本身依赖 lodash，但是默认没有对 lodash 进行导入（认为全局存在 lodash），那么我们就可以通过 ProvidePlugin 来实现 shimming 的效果

webpack 并不推荐随意的使用 shimming

- webpack 背后的整个理念是使前端开发更加模块化
- 也就是说，需要编写具有封闭性的，不存在隐含依赖（比如全局变量）的彼此隔离的模块

```js
module.exports = {
  plugins: [
    new ProvidePlugin({
      // 相当于对全局使用 import ReactDom from 'react-dom/client'
      // ReactDom: 'react-dom/client',
      // 相当于对全局使用 import { createRoot } from 'react-dom/client
      createRoot: ['react-dom/client', 'createRoot']
    })
  ],
}
```



### 对 CSS 文件进行单独提取

提取 css 需要安装对应的依赖：`npm install mini-css-extract-plugin -D` 

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        
        use: [
          // 'style-loader', // style-loader 将会通过 js 在 html 插入 style 标签，并将相应的样式填入（开发阶段）
          MiniCssExtractPlugin.loader, // 提取到单独的 css 文件中，通过 link 进行引入（生产阶段）
          'css-loader'
        ], // loader 将会从后往前进行加载
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './index.html'
    }),
    new MiniCssExtractPlugin({
      // 对打包后的文件名字，如下打包将会放入 css 文件夹中
      filename: 'css/[name]-css.css',
      chunkFilename: 'css/[name].css', // 分包后的包文件名
    })
  ],
}
```



### Hash、ContentHash、ChunkHash

在给打包的文件进行命名的时候，会使用 placeholder，placeholder 中有几个比较常用的属性

hash 本身使通过 MD4 的散列函数处理后，生成一个 128 位的 hash 值（32 个十六进制）

- fullhash：当某个文件发生改变时，所有文件的 hash 值都会重新生成，且一样
  - hash 值的生成和整个项目有关系
  - 当存在两个入口文件 index.js 和 main.js 时，他们分别会输出到不同的 bundle.js 文件中，并且在文件名称中我们有使用 hash
  - 这个时候，如果修改了 index.js 文件中的内容，那么 hash 会发生变化，意味着两个文件的名称都会发生变化
- chunkhash：当某个文件发生改变时，只有改变的文件 hash 值会重新生成
  - 可以有效解决上面的问题，他会根据不同的入口进行解析来生成 hash 值
  - 比如修改了 index.js，那么 main.js 的 chunkhash 是不会发生改变的
  - chunkhash 根据不同的入口文件(entry)进行依赖文件解析、构建对应的chunk，生成对应的哈希值。当某个文件内容发生变动时，再次执行打包，只有该文件以及依赖该文件的文件的打包结果 hash 值会发生改变
- contenthash：表示生成的文件 hash 名称，只和内容有关（推荐）
  - 比如我们的 index.js 引入了一个 style.css，style.css 有被抽取到一个独立的 css 文件
  - 这个 css 文件在命名时，如果我们使用的是 chunkhash，那么当 index.js 文件的内容发生变化时，css 文件的命名也会发生变化
  - 这个时候我们可以使用 contenthash 

### DLL 库

DLL 全程是动态链接库（Dynamic Link Library）

- 它指的是我们可以共享，并且不经常改变的代码，抽取称一个共享的库
- 这个库在之后编译的过程中，会被引入到其他项目的代码中

使用过程：

1. 打包 DLL 库
2. 项目中引入 DLL 库

现在已经不再使用，移除原因：webpack 4 已经提供很好的性能，没有必要再花费时间去维护 DLL

### Terser

- Terser 是一个 JavaScript 的解析（Parser）、Mangler（绞肉机）、Compressor（压缩机） 的工具集

- 早期我们会使用 uglify-js 来压缩、丑化我们的 JavaScript 代码，但是目前已经不再维护，并且不支持 ES6+ 的语法
- Terser 是从 uglify-es fork 过来的，并且保留了他原来的大部分 API 以及适配 uglify-es 和 uglify-js@3 等

#### 命令行使用方式

```bash
terser [filename] -o [outputfilename] -c [arguments] -m [arguments]
```

-c 表示压缩（compress）

- arrows：class 或 object 中的函数，转换成箭头函数
- arguments：将函数中使用的 arguments[index] 转成对应的形参名称
- dead_code：移除不可达的代码（tree shaking）

```bash
npx terser a.js -o a.min.js -c arrows=true,arguments=true,dead_code=true
```

优化前代码

```js
function test(name1, name2) {
  console.log(name1, name2)
  console.log(arguments[0], arguments[1])
}

test('123', '234')

const obj = {
  bar() {
    return 'bar'
  }
}

class Person {
  name = 'zhangsan'
  getName() {
    return 'lisi'
  }
}

if (false) {
  console.log('false')
}
```

优化后代码

```js
function test(name1,name2){console.log(name1,name2),console.log(name1,name2)}test("123","234");const obj={bar:()=>"bar"};class Person{name="zhangsan";getName(){return"lisi"}}
```

-m 选项

- top_level：优化顶层所有变量名
- keep_fnames：保持函数原名

- keep_classnames：保持原类名

```bash
npx terser .\src\main.js -o main.min.js -c arrows=true,arguments=true,dead_code=true -m toplevel=true,keep_fnames=true
```

优化后代码

```js
function test(n,s){console.log(n,s),console.log(n,s)}test("123","234");const n={bar:()=>"bar"};class s{name="zhangsan";getName(){return"lisi"}}
```

#### 在 webpack 中配置 terser

- 在 webpack 中有一个 minimizer 属性，在 production 模式下，默认就是使用 TerserPlugin 来处理我们的代码的

```js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const TerserPlugin = require('terser-webpack-plugin')

module.exports = {
  mode: 'development',
  devtool: false,
  entry: './src/main.js',
  output: {
    filename: 'js/[contenthash:10]-[name].js',
    path: path.resolve(__dirname, './build'),
    clean: true
  },
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        // 屏蔽第三方库中的注释
        extractComments: false,
        // 使用多进程并发提高构建速度，默认值是 true
        // 并发运行的默认数量是 os.cpu().length - 1
        // 我们也可以设置自己的个数，使用默认即可
        parallel: true,
        terserOptions: {
          compress: {
            // 配置函数中使用 arguments 进行优化
            arguments: true,
            // 未被引用的代码将被删除
            unused: true
          },
          // 配置 Mangler（绞肉机）
          mangle: true,
          // 对 mangle 传入的参数
          keep_fnames: true
        }
      })
    ]
  },
  resolve: {
    extensions: ['.css', '.js']
  }
}
```



### CSS 压缩

需要安装对应的插件

```bash
npm install css-minimizer-webpack-plugin -D
```

- css 压缩通常是去除无用的空格等，它使用的是 cssnano 工具来进行优化、压缩 CSS 也可以单独使用

```js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const CSSMinimizerPlugin = require('css-minimizer-webpack-plugin')

module.exports = {
  mode: 'development',
  devtool: false,
  entry: './src/main.js',
  output: {
    filename: 'js/[contenthash:10]-[name].js',
    path: path.resolve(__dirname, './build'),
    clean: true
  },
  optimization: {
    minimize: true,
    minimizer: [
      new CSSMinimizerPlugin()
    ]
  },
  resolve: {
    extensions: ['.css', '.js']
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader'
        ]
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './index.html'
    }),
    new MiniCssExtractPlugin({
      filename: 'style/[name]-[contenthash:10].css'
    })
  ]
}
```

### webpack 配置文件的拆分

在编写 webpack 配置文件时，可以导出一个函数，并且可以通过 --env 给该函数传入相应的参数

```js
// common.config.js
const path = require('path')

const commonConfig = {
  mode: 'development',
  devtool: false,
  entry: './src/main.js',
  output: {
    clean: true,
    filename: '[name]-[contenthash:10].js',
    path: path.resolve(__dirname, '../build')
  }
}

module.exports = function(env) {
  if (env.production) {
    console.log('production environment')
  } else {
    console.log('development environment')
  }
  return commonConfig
}
// package.json
{
  "name": "multiply_config",
  "main": "src/main.js",
  "scripts": {
    "build": "webpack --config ./config/common.config.js --env development"
  },
  "devDependencies": {
    "webpack": "^5.97.1",
    "webpack-cli": "^6.0.1"
  }
}
```

使用 webpack-merge 插件进行拆分

```js
// common.config.js
const path = require('path')
const HTMLWebpackPlugin = require('html-webpack-plugin')
const { merge } = require('webpack-merge')

const production = require('./production.config')
const development = require('./development.config')

const commonConfig = {
  entry: './src/main.js',
  output: {
    clean: true,
    filename: 'js/[name]-[contenthash:10].js',
    path: path.resolve(__dirname, '../build')
  },
  resolve: {
    extensions: ['.css', '.js']
  },
  plugins: [
    new HTMLWebpackPlugin({
      template: './index.html'
    })
  ]
}

module.exports = function(env) {
  if (env.production) {
    console.log('production environment')
    return merge(commonConfig, production)
  } else {
    console.log('development environment')
    return merge(commonConfig, development)
  }
}
// development.config.js
module.exports = {
  mode: 'development',
  devtool: 'source-map',
  module: {
    rules: [
      {
        test: /\.css/,
        use: ['style-loader', 'css-loader']
      }
    ]
  },
}
// production.config.js
const MiniCSSExtractPlugin = require('mini-css-extract-plugin')
const TerserPlugin = require('terser-webpack-plugin')
const CSSMinimizerPlugin = require('css-minimizer-webpack-plugin')


module.exports = {
  mode: 'production',
  devtool: false,
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            arrows: true,
            arguments: true,
          },
          mangle: true,
          toplevel: true,
          keep_fnames: true,
          keep_classnames: true
        }
      }),
      new CSSMinimizerPlugin()
    ]
  },
  module: {
    rules: [
      {
        test: /\.css/,
        use: [MiniCSSExtractPlugin.loader, 'css-loader']
      }
    ]
  },
  plugins: [
    new MiniCSSExtractPlugin({
      filename: 'css/[name]-[contenthash:10].css'
    })
  ]
}

```



### TreeShaking

通过使用 `optimization.userdExports: true` 联合 terser 进行删除不会使用的代码

webpack.config.js

```js
module.exports = {
  mode: 'production',
  devtool: false,
  optimization: {
    minimize: true,
    // 使用该参数，将会在生成的代码中标注那些函数没被使用
    // 通过 terser plugin 对标记的代码进行删除
    // plugin 模式下会自动开启该功能
    usedExports: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            arrows: true,
            arguments: true,
          },
          mangle: true,
          toplevel: true,
          keep_fnames: true,
          keep_classnames: true
        }
      })
    ]
  }
}

```

通过 sideEffects 方式删除多余代码在 package.json 中

```json
{
  "name": "multiply_config",
  // 告诉 webpack 所有文件都没有副作用，可以删除
  // "sideEffects": false,
  // 通过数组的方式告诉 webpack 那些文件存在副作用
  "sideEffects": [
    "./src/utils/sideEffect.js",
    "*.css" // 所有 css 将不会被优化删除
  ],
  "main": "src/main.js",
}

```

在模块中使用类似 `window.a = '123'` 就时副作用代码

#### CSS tree shaking

安装对应依赖：

```bash
npm install purgecss-webpack-plugin -D
```



### Scope Hosting

由于打包之后的文件每个模块会存在一个单独的作用域，在一个模块中使用另一个模块中的函数等将会涉及跨作用域的问题，从而导致的性能低下。

该配置在 production 模式下是默认开启的

```js
const webpack = require('webpack')

module.exports = {
  plugins: [
    new webpack.optimize.ModuleConcatenationPlugin()
  ]
}
```



### HTTP 压缩

HTTP 压缩是一种内置在服务器和客户端之间的，以改进传输速度和带宽利用率的方式

1. 第一步，http 数据在服务器发送前就已经被压缩了；（可以在 webpack 中完成）

2. 第二步，兼容的浏览器在向服务器发送请求时，或告知服务器自己支持哪些压缩格式

   ```http
   GET /encrypted-area HTTP/1.1
   Host: www.example.com
   Accept-Encoding: gzip, deflate
   ```

3. 第三步，服务器在浏览器支持的压缩格式下，直接返回对应的压缩后的文件，并且在响应头中告知浏览器

   ```http
   HTTP/1.1 200 OK
   Content-Encoding: gzip
   ```

### Gzip 压缩

webpack 中相当于是实现了 HTTP 压缩的第一步操作，我们可以使用 CompressionPlugin

- 安装 CompressionPlugin

  ```bash
  npm install compression-webpack-plugin -D
  ```

- 使用 CompressionPlugin

  ```js
  const CompressionPlugin = require('compression-webpack-plugin')
  
  module.exports = {
    plugins: [
      new CompressionPlugin({
        test: /\.(css|js)$/, // 匹配哪些文件将被压缩
        // threshold: 500, // 设置文件从多大开始压缩
        minRatio: 0.7, // 至少压缩的比例
        algorithm: 'gzip' // 采用的压缩算法
      })
    ]
  }
  ```

### HTML 压缩

```js
const path = require('path')
const HTMLWebpackPlugin = require('html-webpack-plugin')
const { merge } = require('webpack-merge')

const production = require('./production.config')
const development = require('./development.config')

const commonConfig = function(isProduction) {
  return {
    entry: './src/main.js',
    output: {
      clean: true,
      filename: 'js/[name]-[contenthash:10].js',
      path: path.resolve(__dirname, '../build')
    },
    resolve: {
      extensions: ['.css', '.js']
    },
    plugins: [
      new HTMLWebpackPlugin({
        // 当文件内容发生改变的时候才重新生成
        cache: true,
        minify: isProduction ? {
          // 压缩时移除注释
          removeComments: true,
          // 删除空属性，例如 <div class=''></div> => <div></div>
          removeEmptyAttributes: true,
          // 移除多以的属性，例如：<input type='text'> => <input>
          removeRedundantAttributes: true,
          // 删除空行和空格
          collapseWhitespace: true,
          // 压缩内联 css
          minifyCSS: true,
          // 压缩内联 js
          minifyJS: {
            mangle: {
              toplevel: true
            }
          },
        } : false,
        template: './index.html'
      })
    ]
  }
}

module.exports = function(env) {
  if (env.production) {
    console.log('production environment')
    return merge(commonConfig(true), production)
  } else {
    console.log('development environment')
    return merge(commonConfig(false), development)
  }
}
```

### 针对于打包过程进行分析

需要安装 speed-measure-webpack-plugin

```bash
npm install speed-measure-webpack-plugin -D
```



### 对打包后的文件进行分析

方式一：添加 `--profile --json=stats.json` 将会生成 stats.json 文件，该文件可通过 webpack 的在线分析平台进行分析

```json
{
  "scripts": {
    "build:dev": "webpack --config ./config/common.config.js --env development --profile --json=stats.json",
    "build:pro": "webpack --config ./config/common.config.js --env production"
  },
}
```

方式二：使用插件进行分析打包后的文件 `webpack-bundle-analyzer` 

```bash
npm install webpack-bundle-analyzer -D
```

在 webpack 中配置该插件

```js
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer')

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin()
  ]
}
```

该方式会自动开启一个端口用于展示生成后的文件状态：localhost:8080 



## Webpack 源码

### createCompiler

webpack 首先会创建 compiler，然后注册插件

```js
/**
 * @param {WebpackOptions} rawOptions options object
 * @param {number} [compilerIndex] index of compiler
 * @returns {Compiler} a compiler
 */
const createCompiler = (rawOptions, compilerIndex) => {
	const options = getNormalizedWebpackOptions(rawOptions);
	applyWebpackOptionsBaseDefaults(options);
	const compiler = new Compiler(
		/** @type {string} */ (options.context),
		options
	);
	new NodeEnvironmentPlugin({
		infrastructureLogging: options.infrastructureLogging
	}).apply(compiler);
	if (Array.isArray(options.plugins)) {
		for (const plugin of options.plugins) {
			if (typeof plugin === "function") {
				// 当插件是一个函数时将会执行这个函数，传入 compiler，并给该函数绑定 compiler
				/** @type {WebpackPluginFunction} */
				(plugin).call(compiler, compiler);
			} else if (plugin) {
				// 当插件时一个对象时，必须存在一个 apply 函数，compiler 将会传入这个函数
				plugin.apply(compiler);
			}
		}
	}
	const resolvedDefaultOptions = applyWebpackOptionsDefaults(
		options,
		compilerIndex
	);
	if (resolvedDefaultOptions.platform) {
		compiler.platform = resolvedDefaultOptions.platform;
	}
	compiler.hooks.environment.call();
	compiler.hooks.afterEnvironment.call();
	new WebpackOptionsApply().process(options, compiler);
	compiler.hooks.initialize.call();
	return compiler;
};
```





### webpack 解析过程



### 自定义 Loader

Loader 是用于对模块的源代码进行转换（处理），Loader 本质上是一个导出为函数的 Js 模块，Loader  runner 库会调用这个函数，然后将上一个 loader 产生的结果或者资源文件传入进去

webpack.config.js

```js
const path = require('path')

module.exports = {
  mode: 'development',
  devtool: false,
  entry: './src/main.js',
  output: {
    clean: true,
    filename: '[name]-[contenthash:6].js',
    path: path.resolve(__dirname, './build')
  },
  resolve: {
    extensions: ['.js']
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          {
            loader: './loaders/loader04.js',
            options: {
              name: 'zhangsan',
              age: 18
            }
          }
        ]
      }
      // {
      //   test: /\.js$/,
      //   use: [
      //     // 可以使用相对路径，loader 将会从后往前进行加载
      //     './loaders/loader01.js',
      //     './loaders/loader02.js',
      //     './loaders/loader03.js',
      //   ]
      // },

      // 如下代码也将会从后往前加载
      // {
      //   test: /\.js$/,
      //   use: './loaders/loader01.js'
      // },
      // {
      //   test: /\.js$/,
      //   use: './loaders/loader02.js'
      // },
      // {
      //   test: /\.js$/,
      //   use: './loaders/loader03.js'
      // }
    ]
  }
}
```

loader01.js

```js
/**
 * 
 * @param {*} content 上一个 loader 返回的结果或者文件的内容
 * @param {*} map 根 source map 有关
 * @param {*} meta 元数据
 * @returns 返回处理后的内容
 */
module.exports = function(content, map, meta) {
  console.log('====================')
  console.log(content)
  console.log('====================')
  return content
}
```

#### callback 函数

```js
/**
 * 
 * @param {*} content 上一个 loader 返回的结果或者文件的内容
 * @param {*} map 根 source map 有关
 * @param {*} meta 元数据
 * @returns 返回处理后的内容
 */
module.exports = function(content, map, meta) {
  // 通过 callback 可以返回信息
  const callback = this.callback
  // 当 return 和 callabck 同时存在时，callback 返回的内容将会传递给下一个 loader
  /**
   * 参数一： 异常信息
   * 参数二：返回给下一个 loader 的内容
   */
  // callback(null, content + '\n//aaaaaaaaaaa')
  setTimeout(() => {
    // 当将 callback 放到异步函数里面，其返回值将不生效
    // 其后面的 loader 将不会延迟执行
    callback(null, content + '\n//aaaaaaaaaaa')
  }, 1000)

  console.log('==========2=========')
  console.log(content)
  console.log('====================')
  return content
}
```

#### async 函数（异步函数）

```js
/**
 * 
 * @param {*} content 上一个 loader 返回的结果或者文件的内容
 * @param {*} map 根 source map 有关
 * @param {*} meta 元数据
 * @returns 返回处理后的内容
 */
module.exports = function(content, map, meta) {
  const callabck = this.async()
  // 通过 async 返回的函数作为异步函数，该 loader 将被视为一个异步 loader
  setTimeout(() => {
    // 下一个 loader 的 content 参数将会加上这个 loader 返回的结果，其优先级大于 return，return 返回的东西将不会生效
    // 其后面的 loader 将会延后执行
    callabck(null, content + '\n//3333333')
  }, 1000)
  console.log('==========3=========')
  console.log(content)
  console.log('====================')
  return content
}
```

#### 给 loader 传递参数

```js
const { getOptions } = require("loader-utils")

module.exports = function(content) {
  // 方式一：通过 loader-utils （webpack 开发）的库来获取
  console.log(getOptions(this)) // 现在执行将会报错 TypeError: getOptions is not a function
  // 方式二：通过 this.getOptions() 函数来获取
  console.log(this.getOptions()) // { name: 'zhangsan', age: 18 }
  return content
}
```

#### 对传入的参数进行校验

需要使用第三方库的支持

```bash
npm install schema-utils -D
```

loader.js

```js
const { validate } = require('schema-utils')

module.exports = function(content) {
  const options = this.getOptions()

  /**
   * 参数一：校验的规则
   * 参数二：需要校验的数据
   */
  validate({
    type: "object",
    properties: {
      "username": {
        type: 'string'
      },
      'age': {
        type: 'number'
      }
    }
  }, options)

  return content
}
```

#### markdown loader

```js
const { marked } = require('marked')
const hljs = require('highlight.js')


const renderer = {
  code(obj) {
    /*
    {
      type: 'code',
      raw: '```javascript\n' +
        "const a = '213'\n" +
        'let obj = {\n' +
        '  age: 18\n' +
        '}\n' +
        "console.log('aaa')\n" +
        '```',
      lang: 'javascript',
      text: "const a = '213'\nlet obj = {\n  age: 18\n}\nconsole.log('aaa')"
    }
    */
    console.log(obj)
    const language = hljs.getLanguage(obj.lang) ? obj.lang : 'plaintext';
    try {
      const highlightedCode = hljs.highlight(obj.text, { language }).value;
      return `<pre><code class="hljs ${language}">${highlightedCode}</code></pre>`;
    } catch (__) {
      return `<pre><code class="hljs ${language}">${obj.text}</code></pre>`;
    }
  }
};

module.exports = function (content) {
  marked.use({ renderer: renderer })
  const htmlContent = marked(content)
  const moduleContent = `var md = \`${htmlContent}\`; export default md`

  return moduleContent
}
```



### 自定义插件

webpack 中的compiler 和 compilation 通过注入插件的方式，来监听 webpack 的声明周期，其创建了 Tapable 库中的各种 Hook 的实例

1. webpack 函数中的 createCompiler 方法中，注册了所有的插件
2. 在注册插件时，会调用插件函数或者插件对象的 apply 方法
3. 插件方法会接受 compiler 对象，我们可以通过 compiler 对象来注册 Hook 事件
4. 某些插件也会传入一个 compilation 的对象，我们也可以监听 compilation 的 hook 事件

#### Tapable

Tapable 是管理着需要的 Hook，这些 Hook 可以被应用到我们的插件中

- bail：当有返回值时，就不会执行后续的事件触发了
- loop：当返回值为 true，就会反复执行该事件，当返回值为 undefined 或者不返回内容时，就退出事件
- waterfall：当返回值不为 undefined 时，会将这次放回的结果作为下次事件的第一个参数
- parallel：并行，会同时执行事件处理回调结束，不会等到这个事件执行结束才执行下一次事件处理回调
- series：串行，会等待上一次异步的 Hook

官方提供的 Hook

- 同步 Hook：SyncHook、SyncBailHook、SyncWatefallHook、SyncLoopHook
- 异步 Hook，两个事件处理回调，不会等待上一次处理回调结果后再执行下一次回调
  - Paralle（并行）：AsyncPralleHook、AsyncParalleBailHook
  - Series（串行）：AsyncSeriesHook、AsyncSeriesBailHook、AsyncSeriesWaterfallHook

安装 tapable 库

```bash
npm install tapable -D
```

基本使用

```js
const { SyncHook } = require('tapable')

class Compiler1 {
  constructor() {
    this.hooks = {
      // 定义 hook， name，age 为调用这个 hook 需要传递的参数
      syncHook: new SyncHook(['name', 'age'])
    }

    // 使用 hook 监听事件
    this.hooks.syncHook.tap('event1', (name, age) => {
      console.log('hook1 execute', name, age)
    })

    this.hooks.syncHook.tap('event2', (name, age) => {
      console.log('hook1 execute 2', name, age)
    })
  }
}

const compiler = new Compiler1()
compiler.hooks.syncHook.call('zhangsan', 18)
```

bail

```js
const { SyncBailHook } = require('tapable')

class Compiler1 {
  constructor() {
    this.hooks = {
      // 定义 hook， name，age 为调用这个 hook 需要传递的参数
      bailHook: new SyncBailHook(['name', 'age'])
    }

    // 使用 hook 监听事件
    this.hooks.bailHook.tap('event1', (name, age) => {
      console.log('hook1 execute', name, age)
      // 当存在返回值时，后续的回调将不会执行
      return 123
    })

    this.hooks.bailHook.tap('event2', (name, age) => {
      console.log('hook1 execute 2', name, age)
    })
  }
}

const compiler = new Compiler1()
compiler.hooks.bailHook.call('zhangsan', 18)
```

loop

```js
const { SyncLoopHook } = require('tapable')

class Compiler1 {
  constructor() {
    this.hooks = {
      // 定义 hook， name，age 为调用这个 hook 需要传递的参数
      loopHook: new SyncLoopHook(['name', 'age'])
    }

    let count = 3

    // 使用 hook 监听事件
    this.hooks.loopHook.tap('event1', (name, age) => {
      console.log('hook1 execute', name, age)
      // 当不存在返回值或者返回值为 undefined 时将会中断循环
      return count -- === 0 ? undefined : count
    })

    this.hooks.loopHook.tap('event2', (name, age) => {
      console.log('hook1 execute 2', name, age)
    })
  }
}

const compiler = new Compiler1()
compiler.hooks.loopHook.call('zhangsan', 18)
```

waterfall

```js
const { SyncWaterfallHook } = require('tapable')

class Compiler1 {
  constructor() {
    this.hooks = {
      // 定义 hook， name，age 为调用这个 hook 需要传递的参数
      waterfallHook: new SyncWaterfallHook(['name', 'age'])
    }

    let count = 3

    // 使用 hook 监听事件
    this.hooks.waterfallHook.tap('event1', (name, age) => {
      console.log('hook1 execute', name, age)
      // 返回值不为 undefined 时，将作为下一个回调函数第一个参数
      return { gender: 'male' }
    })

    this.hooks.waterfallHook.tap('event2', (name, age) => {
      console.log('hook1 execute 2', name, age) // hook1 execute 2 { gender: 'male' } 18
    })
  }
}

const compiler = new Compiler1()
compiler.hooks.waterfallHook.call('zhangsan', 18)
```

parallel

```js
const { AsyncParallelHook } = require('tapable')

class Compiler1 {
  constructor() {
    this.hooks = {
      // 定义 hook， name，age 为调用这个 hook 需要传递的参数
      parallelHook: new AsyncParallelHook(['name', 'age'])
    }

    // 使用 hook 监听事件
    this.hooks.parallelHook.tapAsync('event1', (name, age) => {
      setTimeout(() => {
        console.log('hook1 execute', name, age)
      }, 1000)
    })

    this.hooks.parallelHook.tapAsync('event2', (name, age) => {
      setTimeout(() => {
        console.log('hook1 execute 2', name, age)
      }, 900)
    })
  }
}

const compiler = new Compiler1()
compiler.hooks.parallelHook.callAsync('zhangsan', 18)
// 输出结果
// hook1 execute 2 zhangsan 18
// hook1 execute zhangsan 18
```

series

```js
const { AsyncSeriesHook } = require('tapable')

class Compiler1 {
  constructor() {
    this.hooks = {
      // 定义 hook， name，age 为调用这个 hook 需要传递的参数
      serieslHook: new AsyncSeriesHook(['name', 'age'])
    }

    // 使用 hook 监听事件
    this.hooks.serieslHook.tapAsync('event1', (name, age, callback) => {
      setTimeout(() => {
        console.log('hook1 execute', name, age)
        // 调用 callback 下面的任务才会执行
        callback()
      }, 1000)
    })

    this.hooks.serieslHook.tapAsync('event2', (name, age, callback) => {
      setTimeout(() => {
        console.log('hook1 execute 2', name, age)
        callback()
      }, 900)
    })
  }
}

const compiler = new Compiler1()
compiler.hooks.serieslHook.callAsync('zhangsan', 18, () => {
  console.log('all tasks finished')
})
// 输出结果
// hook1 execute zhangsan 18
// hook1 execute 2 zhangsan 18
// all tasks finished
```



## Babel

> babel 是一个工具链，主要用于就浏览器或者环境中的 ECMAScript2015 + 代码转化为向后兼容版本的 JavaScript
>
> - 包括：语法转换、源代码转换、Polyfill 实现目标环境缺少的功能等
> - 需下载 @babel/core @babel/cli

在使用 babel 命令时可以设置 plugins 参数，将其转换为使用该插件后的代码：

转化箭头函数为普通函数

```bash
# transform arrow function to function
npm install @babel/plugin-transform-arrow-functions -D
npx babel src --out-dir dist --plugin@babel/plugin-transform-arrow-functions
```

转化块级作用域

```bash
# transform block scoping
npm install @babel/plugin-transform-block-scoping -D
npm babel src --out-dir dist --plugin@babel/plugin-transform-block-scoping,@babel/plugin-transform-arrow-functions
```

### 解析原理

https://github.com/jamiebuilds/the-super-tiny-compiler

![1735832299456](.\imgs\babel)

1. 解析阶段（Parsing）
2. 转换阶段（Transformation）
3. 生成阶段（Code Generation）



### 预设

安装 @babel/preset-env

```bash
npm install @babel/preset-env -D

npx babel ./src --out-dir ./dist --presets=@babel/preset-env
```

### babel 配置文件

可以将 babel 的配置信息编写到一个文件中

- babel.config.json（或 .js, .cjs, .mjs）文件（推荐）
  - 可以直接作用于 Monorepos 项目的子包，更加推荐
- .babelrc.json （或 .babelrc, .js, .cjs, .mjs）文件
  - 早期使用较多的配置方案，但是对于配置 Monorepos 项目是比较麻烦的

```js
// babel.config.js
module.exports = {
  presets: [
    ['@babel/preset-env', {
      
    }]
  ]
}
```





## Browserslist 插件

Browserslist 可以在不同的的前端工具之间，共享目标浏览器和 Node.js 版本的配置

条件查询使用的是 caiuse-lite 的工具，这个工具的数据来自于 caniuse 的网站上

browserslist 编写规则：

- defaults：browserslist 的默认浏览器（>0.5%, last 2 versions, Firefox ESR, not dead）
- 5%：通过全局使用情况统计信息选择的浏览器版本，>=, < 和 <=

dead：24 个月内没有官方支持或更新的浏览器

last 2 versions：每个浏览器的最后两个版本

### 使用方式

1. 可以在 package.json 文件中编写

```json
"browserlists": [
  "last 2 versions",
  "not dead",
  "> 0.2%"
]
```

2. 在 .browserslistrc 文件中编写（最常使用）
   1. 该配置将会在多个工具之间进行共享（postcss/babel）

```markdown
> 0.2%
last 2 versions
not dead
```

3. 可以在 webpack.config.js 文件中配置（不常用）

```js

module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.js$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              ['@babel/preset-env', {
                targets: '>5%', // 将会覆盖 browserslist 文件中的内容
              }]
            ]
          }
        }
      }
    ]
  }
}
```



## Polyfill

将一些高版本 js 代码对应于低版本中不存在的 API（Promise，string.includes 等函数）采用补丁的方式进行替换

安装对应依赖（开发和生产环境都需要依赖）

```bash
npm install core-js regenerator-runtime
```

useBuiltIns 属性的可选值：

1. false：
   - 打包后的文件不适用 polyfill 来进行适配
   - 并且这个时候是不需要设置 corejs 属性的
2. usage：
   - 会根据源代码中出现的语言特性，自动检测所需要的 polyfill
   - 这样可以确保最终包离的 polyfill 数量的最小化，打包的包相对较小
   - 可以设置 corejs 属性来确定使用的 corejs 的版本
3. entry
   - 如果我们依赖的某一个库本身使用了 polyfill 的特性，但是因为我们使用的是 usage，所以之后用户浏览器可能会报错，如果担心这种情况，可以使用 entry
   - 并且需要在入口文件中添加 `import 'core-js/stable'; import 'regenerator-runtime/runtime';` 
   - 这样做会根据 browserslist 目标导入所有的 polyfill，但是对应的包会很大

```js
module.exports = {
  presets: [
    ['@babel/preset-env', {
      corejs: 3,
      useBuiltIns: 'entry' // 表示不使用 polyfill，可选值：【false|usage(使用polyfill进行填充)|entry】
    }]
  ]
}
```

入口文件 main.js

```js
import 'core-js/stable'
import 'regenerator-runtime/runtime'

// ...
```



## CDN 服务器

CDN 称之为内容分发网络（Content Delivery Network 或 Content Distribution Network）

- 它是指通过相互连接的网络系统，利用最靠近每个用户的服务器
- 更快、更可靠的将音乐、图片、视频、应用程序以及其他文件发送给用户
- 来提供高性能、可扩展性及低成本的网络内容传递给用户

在开发中，我们使用 CDN 主要是两种方式

1. 打包所有的静态资源，放到 CDN 服务器，用户所有资源都是通过 CDN 服务器加载的

   ```js
   module.exports = {
     output: {
       filename: '[name]-bundle.js',
       chunkFilename: '[name]_chunk.js',
       // path 必须指定绝对路径
       path: path.resolve(__dirname, './build'),
       clean: true, // when rebuild, the output dir will be cleaned
       publicPath: 'https://xxxcdn.com', // 设置 CDN 服务器域名
     },
   }
   ```

   

2. 一些第三方资源放到 CDN 服务器上

   国内 CDN 平台 [BOOTCDN](https://www.bootcdn.cn/) 

   在 模板 index 中直接引入需要从 CDN 引用的第三方库

   ```html
     <script src="https://cdn.bootcdn.net/ajax/libs/react/18.3.1/umd/react.production.min.js"></script>
   ```

   在 webpack 中配置相应从 CDN 引用的包

   ```js
   module.exports = {
     externals: {
       // key: 排除框架的名称，import xxx from 'key'
       // value：从 CDN 请求下来的 js 中提供的对应名称
       react: 'React',
       // react_dom: 'ReactDom'
     },
   }
   ```



## Gulp

gulp 的核心理念是 task runner

- 可以定义自己的一系列任务，等待任务被执行
- 基于文件 stream 的构建流
- 我们可以使用 gulp 的插件体系来完成某些任务

webpack 的核心理念是 module bundler

- webpack 是一个模块化的打包工具
- 可以使用各种各样的 loader 来加载不同的模块
- 可以使用各种各样的插件在 webpack 打包的生命周期完成其他任务

gulp 相对于 webpack 的优缺点

- gulp 相对于 webpack 思想更加的简单、医用，更适合编写一些自动化的任务
- 但是目前对于大型项目（Vue、React、Angular）并不会使用 gulp 来构建，比如默认 gulp 是不支持模块化的

### 基本使用

安装对应依赖

```bash
npm install gulp
```

编写 gulpfile 文件

```js
const gulp = require('gulp')


// 使用 npx gulp foo1 执行
const foo1 = (callback) => {
  console.log('task 1 finished')
  // 需要调用 callback 才能知道该任务已完成
  callback()
}

// 执行方式二
gulp.task('foo2', (callback) => {
  console.log('task 2 finished')
  callback()
})

module.exports = {
  foo1
}

// 默认任务 通过 npx gulp
module.exports.default = (callback) => {
  console.log('default task finished')
  callback()
}
```

### 创建 gulp 任务

每个 gulp 任务都是一个异步的 JavaScript 代码

- 此函数可以接收一个 callback 作为参数，调用 callback 函数，那么该任务会结束
- 或者是一个返回 stream、promise、event emitter、child process 或 observable 类型的函数

任务可以是 public 或者 private 类型的

- 公开任务（public tasks）从 gulpfile 中被导出（export），可以通过 gulp 命令直接调用
- 私有任务（private tasks）被设计为在内部使用，通常作为 series() 或 parallel() 组合的组成部分

*在 gulp 4 之前，注册任务时需要通过 gulp.task 的方式进行注册，也即上面的方式二*

### 多任务

```js
const { series, parallel } = require('gulp')


// 使用 npx gulp foo1 执行
const foo1 = (callback) => {
  setTimeout(() => {
    console.log('foo1')
    callback()
  }, 3000)
}

const foo2 = (callback) => {
  setTimeout(() => {
    console.log('foo2')
    callback()
  }, 2000)
}

const foo3 = (callback) => {
  setTimeout(() => {
    console.log('foo3')
    callback()
  }, 1000)
}
// 任务会依次执行
const seriesTask = series(foo1, foo2, foo3)
// 任务将并行执行
const parallelTask = parallel(foo1, foo2, foo3)

module.exports = {
  seriesTask,
  parallelTask
}
```

### 读写文件

gulp 暴露了 src 和 dest 函数，用于处理计算机存放的文件

- src 接收一个正则路径，并从文件系统中读取文件然后生成一个 Node 流（Stream），它将所有匹配的文件放入内存中并通过读取流（Stream进行处理）
- 由 src 产生的流（Stream）应当从任务（task 函数）中返回并发出异步完成的信号
- dest 接收一个输出目录作为参数，并且它还会产生一个 Node 流（Stream），通过该流将文件内容输出到文件中

pipe 方法接收一个转换流（Transform stream）或 可写流（Writable Stream）

转换流或可写流，拿到数据之后可以对数据进行操作，再次传递给下一个转换流或可写流

```js
const { src, dest } = require('gulp')

const copyFile = () => {
  return src('./src/**').pipe(dest('./dist'))
}

module.exports = {
  copyFile
}
```

### 对 Js 文件进行转化和压缩

```js
const { src, dest, watch } = require('gulp')
const babel = require('gulp-babel')
const terser = require('gulp-terser')

const zipFile = () => {
  return src('./src/**')
    .pipe(babel({ presets: ['@babel/preset-env']}))
    // .pipe(terser({ mangle: { toplevel: true }}))
    .pipe(terser({ toplevel: true }))
    .pipe(dest('./dist'))
}

// 设置自动监听
watch('./src/**/*js', zipFile)

module.exports = {
  zipFile
}
```

### 搭建项目

安装依赖

```bash
npm install gulp-babel gulp-terser gulp-htmlmin gulp-less gulp-inject -D
```

html 文件

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <!-- inject:css -->
  <!-- endinject -->
</head>
<body>
  <!-- inject:js -->
  <!-- endinject -->
</body>
</html>
```

gulpfile.js

```js
const { src, dest, series, parallel, watch } = require('gulp')

const babel = require('gulp-babel')
const terser = require('gulp-terser')

const less = require('gulp-less')
const inject = require('gulp-inject')

const htmlmin = require('gulp-htmlmin')
const browserSync = require('browser-sync')

const jsTask = () => {
  return src('./src/**/*.js')
    .pipe(babel({ presets: ['@babel/preset-env'] }))
    .pipe(terser({ mangle: { toplevel: true }}))
    .pipe(dest('./dist/js'))
}

const cssTask = () => {
  return src('./src/**/*.less')
    .pipe(less())
    .pipe(dest('./dist/css'))
}

const htmlTask = () => {
  return src('./src/**/*.html')
    .pipe(htmlmin())
    .pipe(dest('./dist'))
}

// 注入时使用相对路径设置 relative 为 true
const injectTask = () => {
  return src('./dist/**/*.html')
    .pipe(inject(src(['./dist/**/*.js', './dist/**/*.css']), { relative: true}))
    .pipe(dest('./dist'))
}

// 开启本地服务
const bs = browserSync.create()
const serveTask = () => {
  bs.init({
    port: 8000,
    open: true,
    files: './dist/*',
    server: {
      baseDir: './dist'
    }
  })
}

const parallelTask = parallel(jsTask, cssTask, htmlTask)
const buildTask = series(parallelTask, injectTask)

watch('./src/**/*', buildTask)

const serve = series(buildTask, serveTask)

module.exports = {
  serve,
  buildTask
}
module.exports.default = buildTask
```



## Rollup

Rollup 是一个 JavaScript 的模块化打包工具，可以帮助我们编译小的代码到一个大的，复杂的代码中，比如一个库一个应用程序

### rollup VS webpack

- rollup 是一个模块化的打包工具，但是 rollup 主要是针对 ESModule 进行打包的
- 另外 webpack 通常可以通过各种 loader 处理各样的文件，以及他们之间的依赖关系
- rollup 更多时候是专注于处理 JavaScript 代码的（当然也可以是 css、font、vue 等文件）
- 另外 rollup 的配置和理念相对于 webpack 来说，更加的简洁和容易理解
- 在早期 webpack 不支持 tree shaking 时，rollup 具备更强的优势

通常在实际项目开发过程中，我们都会使用 webpack（比如 react、angular 项目都是基于 webpack）

在对库文件进行打包时，我们通常会使用 rollup （比如 vue、react、dayjs 源码本身都是基于 rollup的，vite 底层使用 rollup）

### rollup 基本使用

安装 rollup

```bash
npm install rollup -D
```

使用命令进行打包

```bash
# 打包为 node 环境
npx rollup ./src/main.js -f cjs -o dist/bundle.cjs
# 打包为浏览器环境
npx rollup ./src/main.js -f iife -o dist/bundle.cjs
# 打包为 amd 环境
npx rollup ./src/main.js -f cjs -o dist/bundle.cjs
# 打包为所有环境都支持，此时必须要指定名字
npx rollup ./src/main.js -f umd --name foo -o dist/bundle.cjs
```

原始代码内容

```js
const foo = (name) => {
  console.log('hello', name)
}

foo('zhangsan')

export { foo }
```



打包后的代码结构

```js
(function (global, factory) {
  typeof exports === 'object' && typeof module !== 'undefined' ? factory(exports) :
  typeof define === 'function' && define.amd ? define(['exports'], factory) :
  (global = typeof globalThis !== 'undefined' ? globalThis : global || self, factory(global.foo = {}));
})(this, (function (exports) { 'use strict';

  const foo = (name) => {
    console.log('hello', name);
  };

  foo('zhangsan');

  exports.foo = foo;

}));

// 对上述代码的解析
// 打包后的代码定义了一个函数，该函数接收两个参数 global 和 factory，factory 为函数
const foo = function (global, factory) {
  // 判断是否存在 exports，并且 module 存在，则将 exports 传入
  typeof exports === 'object' && typeof module !== 'undefined' ? factory(exports) :
  typeof define === 'function' && define.amd ? define(['exports'], factory) :
  // 在浏览器中会存在 global 和 globalThis 变量指向 window，相当于传入 factory为 window.foo = {}
  (global = typeof globalThis !== 'undefined' ? globalThis : global || self, factory(global.foo = {}));
}

// 将 this 和 另外一个函数 传入该函数，执行该函数后，将会对全局对象绑定对应的属性
foo(this, (function (exports) { 'use strict';

  const foo = (name) => {
    console.log('hello', name);
  };

  foo('zhangsan');

  // exports 将为对应环境中的 this（浏览器环境中 window，globalThis，this） 或 exports（node 环境中）
  exports.foo = foo;

}))
```

编写相应配置文件，通过 `npx rollup -c` 执行

```js
module.exports = {
  // 入口文件
  input: './src/main.js',
  output: {
    // 打包后的格式
    format: 'umd',
    // 当使用 umd 时，需要执行名称
    name: 'foo',
    // 输出的文件
    file: './dist/bundle.umd.js'
  }
}
```

对项目进行打包

一般情况下不需要打包 node_modules 文件中的内容，用户可以通过依赖进行安装

```js
// 由于 loadsh 是通过 commonjs 导出的，默认情况下，rollup 不会对其进行打包
// 需要安装 @rollup/plugin-commonjs 解决 使用 esmodule 方式导入 commonjs 包
// 安装 @rollup/plugin-node-resolve 解决 打包 node_modules 文件中的内容

import _ from 'loadsh'

const foo = (name) => {
  console.log(_.join([1, 2, 3]))
  console.log('hello', name)
}

foo('zhangsan')

export { foo }
```

rollup.config.js

```js
// 使用该插件可以通过 commonjs 导出，使用 esmodule 方式导入
const commonjs = require('@rollup/plugin-commonjs')
// 解决 打包 node_modules 文件中的内容
const nodeResolve = require('@rollup/plugin-node-resolve')

module.exports = {
  // 入口文件
  input: './src/main.js',
  output: {
    // 打包后的格式
    format: 'umd',
    // 当使用 umd 时，需要执行名称
    name: 'foo',
    // 输出的文件
    file: './build/bundle.umd.js',
    globals: {
      loadsh: '_'
    }
  },
  plugins: [
    commonjs(),
    nodeResolve()
  ]
}
```

对代码进行转化并压缩

```js
// 使用该插件可以通过 commonjs 导出，使用 esmodule 方式导入
const commonjs = require('@rollup/plugin-commonjs')
// 解决 打包 node_modules 文件中的内容
const nodeResolve = require('@rollup/plugin-node-resolve')
// 对代码进行转换
const {babel} = require('@rollup/plugin-babel')
// 对代码进行压缩
const terser = require('@rollup/plugin-terser')

module.exports = {
  // 入口文件
  input: './src/main.js',
  output: {
    // 打包后的格式
    format: 'umd',
    // 当使用 umd 时，需要执行名称
    name: 'foo',
    // 输出的文件
    file: './build/bundle.umd.js',
    globals: {
      loadsh: '_'
    }
  },
  plugins: [
    commonjs(),
    // nodeResolve(),
    babel({
      presets: ['@babel/preset-env']
    }),
    terser()
  ]
}
```

### 处理 CSS 文件

安装对应的插件

```bash
npm install rollup-plugin-postcss postcss-preset-env -D
```

配置 rollup

```js
// 使用该插件可以通过 commonjs 导出，使用 esmodule 方式导入
const commonjs = require('@rollup/plugin-commonjs')
// 解决 打包 node_modules 文件中的内容
const nodeResolve = require('@rollup/plugin-node-resolve')
// 对代码进行转换
const {babel} = require('@rollup/plugin-babel')
// 对代码进行压缩
const terser = require('@rollup/plugin-terser')
const postcss = require('rollup-plugin-postcss')

module.exports = {
  // 入口文件
  input: './src/main.js',
  output: {
    // 打包后的格式
    format: 'umd',
    // 当使用 umd 时，需要执行名称
    name: 'foo',
    // 输出的文件
    file: './build/bundle.umd.js',
    globals: {
      loadsh: '_'
    }
  },
  plugins: [
    commonjs(),
    // nodeResolve(),
    babel({
      presets: ['@babel/preset-env']
    }),
    terser(),
    postcss({
      plugins: [require('postcss-preset-env')]
    })
  ]
}
```

### 对 vue 进行打包

```js
// 使用该插件可以通过 commonjs 导出，使用 esmodule 方式导入
const commonjs = require('@rollup/plugin-commonjs')
// 解决 打包 node_modules 文件中的内容
const nodeResolve = require('@rollup/plugin-node-resolve')
// 对代码进行转换
const {babel} = require('@rollup/plugin-babel')
// 对代码进行压缩
const terser = require('@rollup/plugin-terser')

const postcss = require('rollup-plugin-postcss')
const vue = require('rollup-plugin-vue')
// 由于 vue 中使用了 node 环境中的 process 去判断当前的环境是否 production 还是 development
// 所以需要执行该变量 process.env.NODE_ENV
const replace = require('rollup-plugin-replace')

module.exports = {
  // 入口文件
  input: './src/main.js',
  output: {
    // 打包后的格式
    format: 'umd',
    // 当使用 umd 时，需要执行名称
    name: 'foo',
    // 输出的文件
    file: './build/bundle.umd.js',
    globals: {
      loadsh: '_',
      vue: 'vue'
    }
  },
  plugins: [
    // commonjs(),
    nodeResolve(),
    babel({
      babelHelpers: 'bundled',
      exclude: /node_modules/,
      presets: ['@babel/preset-env']
    }),
    // terser(),
    postcss({
      plugins: [require('postcss-preset-env')]
    }),
    vue(),
    replace({
      'process.env.NODE_ENV': '"development"'
    })
  ]
}
```

### 搭建本地开发服务

安装对应依赖

```bash
# 安装 本地服务
npm install rollup-plugin-server -D
# 监听文件发生变化重新打包
npm install rollup-plugin-livereload -D
```

配置环境

```js
// 使用该插件可以通过 commonjs 导出，使用 esmodule 方式导入
const commonjs = require('@rollup/plugin-commonjs')
// 解决 打包 node_modules 文件中的内容
const nodeResolve = require('@rollup/plugin-node-resolve')
// 对代码进行转换
const {babel} = require('@rollup/plugin-babel')
// 对代码进行压缩
const terser = require('@rollup/plugin-terser')

const html = require('@rollup/plugin-html')
const postcss = require('rollup-plugin-postcss')
const vue = require('rollup-plugin-vue')
// 由于 vue 中使用了 node 环境中的 process 去判断当前的环境是否 production 还是 development
// 所以需要执行该变量 process.env.NODE_ENV
const replace = require('rollup-plugin-replace')

const server = require('rollup-plugin-server')
const livereload = require('rollup-plugin-livereload') 

module.exports = {
  // 入口文件
  input: './src/main.js',
  output: {
    // 打包后的格式
    format: 'umd',
    // 当使用 umd 时，需要执行名称
    name: 'foo',
    // 输出的文件
    file: './build/bundle.umd.js',
    globals: {
      loadsh: '_',
      vue: 'vue'
    }
  },
  plugins: [
    // commonjs(),
    nodeResolve(),
    babel({
      babelHelpers: 'bundled',
      exclude: /node_modules/,
      presets: ['@babel/preset-env']
    }),
    // terser(),
    postcss({
      plugins: [require('postcss-preset-env')]
    }),
    // html({
    //   include: './index.html'
    // }),
    vue(),
    replace({
      'process.env.NODE_ENV': '"development"'
    }),
    server({
      port: 8000,
      contentDir: '.'
    }),
    livereload()
  ]
}
```

执行命令

```bash
# -w: watch
npx rollup -c -w
```

在执行 rollup 命令时，可以指定参数

```bash
# 通过设定 environment 设置环境变量，可以在 rollup.config.js 文件中 通过 process.env.NODE_ENV 拿到该变量
npx rollup --environment NODE_ENV:production
npx rollup --environment NODE_ENV:development
```



## Vite

由于现在浏览器支持 esmodule 模块化的加载，在开发时，可以直接使用这个特性。但是仍存在如下的问题：

- 在加载文件时，后缀名不能省略
- 加载别的文件时，该文件的依赖也将会在浏览器中进行下载，导致浏览器下载了很多个文件，占用了很大的带宽
- 不支持 ts、vue 等代码文件

vite 对 css、ts 原生支持。对于 less 的使用，只需要安装 less 插件即可



### 配置对 vue 的支持

需要安装配置 vue 的插件 @vite/plugin-vue

```bash
npm install @vite/plugin-vue
```

配置 vite

```js
const { defineConfig } = require('vite')
const vue = require('@vitejs/plugin-vue')

module.exports = defineConfig({
  plugins: [
    vue()
  ]
})
```



## 自定义 CLI

在项目根目录下创建 bin 文件夹，文件夹里面存放 js 代码，在文件头部需要标明执行该文件的环境路径

```js
#!D:/Dahui/Environment/Node/node

console.log('test')
```

创建 package.json 文件，并配置 bin ，其中 demo01 表示执行的命令

```json
{
  "name": "cli",
  "version": "1.0.0",
  "main": "index.js",
  "bin": {
    "demo01": "bin/test.js"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "description": ""
}
```

通过 npm link 命令创建一个连接，该命令将会将配置的 bin 生成一个 demo01（linux），demo01.cmd（cmd） 和 demo01.ps1（powershell） 文件，该文件被存放到 node_global 文件夹中，用于执行该命令

```bash
npm link
```

通过使用 commander 工具来解析传入的参数

```js
#!D:/Dahui/Environment/Node/node

const { program } = require('commander')

// console.log('test')

/**
 * [
 *   'D:\\Dahui\\Environment\\Node\\node.exe', // node 路径
 *   'D:\\Dahui\\Environment\\Node\\node-v18.14.0-win-x64\\node_global\\node_modules\\cli\\bin\\test.js', // 脚本路径
 *   '--version' // 参数
 * ]
 */
// console.log(process.argv)

const version = require('../package.json').version

program.version(version, '-V, --version')
// <valuename> 用于传值 valuename 用于表明传值到哪个属性中
program.option('-d, --destination <destination>', 'where to put the result')

// 监听 --help 参数，当输入该参数将会执行里面的回调函数
program.on('--help', () => {
  console.log('')
  console.log('others')
  console.log('  1111111')
  console.log('  2222222')
})

program.parse(process.argv)

// 获取对 destination 传入的值
console.log(program.opts().destination)
```

### 封装自定义命令，并从远程仓库下载

```js
#!/usr/bin/env node

const { program } = require('commander')
const { useHelp } = require('./core/help')

useHelp()

program.parse(process.argv)
```

help.js

```js
const { program } = require('commander')
const { createFromTemplate, createComponent } = require('./action')

function useHelp() {
  program
    .option('-d, --destination <destination>', 'the destination of direcotry')
    .option('-r, --remove', 'remove the files')

  program
    .command('create <template> [args]')
    .description('the template of project to create')
    .action(createFromTemplate)

  program
    .command('create-cpn <component> [...args]')
    .description('createt a component from template')
    .action(createComponent)
}


module.exports = {
  useHelp
}
```

action.js

```js
const { promisify } = require('util')
// 通过 promisify 使 download 变为一个 promise
const download = promisify(require('download-git-repo'))
const { program } = require('commander')
// 添加模板编译库
const ejs = require('ejs')
const path = require('path')
const fs = require('fs')

async function executeCommand(command, args, opts) {
  return new Promise((resolve) => {    
    const { spawn } = require('child_process')
    // 开启子进程
    const childProcess = spawn(command, args, opts)
    // 将输出传入到主进程的输出
    childProcess.stdout.pipe(process.stdout)
    childProcess.stderr.pipe(process.stderr)

    childProcess.on('close', () => {resolve()})
  })
}

async function createFromTemplate(template, args) {
  console.log(template, args)
  /**
   * 参数一：下载地址 需要指定下载的分支
   * 参数二：下载到哪里
   * 参数三：可选参数，设置为 clone 方式
   */
  await download('direct:git@github.com:DaHui-BT/scoped-css-webpack-plugin.git#main', template, { clone: true })
  let command = 'npm'
  // 判断当前平台，win 需要添加 .exe
  console.log(process.platform)
  if (process.platform == 'win32') {
    command += '.exe'
  }
  // 自动安装相关的依赖
  await executeCommand(command, ['install'], { cwd: `./${template}`})
  // 自动运行该项目
  await executeCommand(command, ['run', 'dev'], { cwd: `./${template}`})
}

async function createComponent(component, args) {
  await new Promise((resolve, reject) => {
    const templatePath = path.resolve(__dirname, './template/template.vue.ejs')
    // 编译模板，并传入需要替换的字符
    ejs.renderFile(templatePath, { name: component, lowerName: component.toLowerCase() }, (err, result) => {
      if (err) {
        reject(err)
      } else {
        resolve(result)
      }
    })
  }).then(async res => {
    const destination = program.opts().destination || './'
    const filePath = path.resolve(__dirname, destination)
    // 判断文件路径是否存在，不存在则创建
    if (!fs.existsSync(filePath)) {
      fs.mkdirSync(filePath)
    }
    // 同步写入文件
    await fs.promises.writeFile(`${filePath}/${component}.vue`, res)
  })
}

module.exports = {
  createFromTemplate,
  createComponent
}
```





# [TODO](https://www.aliyundrive.com/drive/file/all/backup/66f25f1481f8a5c9b2ec48bdb4953a2c7483f60a)

802 day134_Rollup-Vite打包与原理-脚手架开发_14_(掌握)vite-esbuild构建工具的原理解析.mp4 

