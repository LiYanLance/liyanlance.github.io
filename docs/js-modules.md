# JS 模块化

刚接触前端的时候, 看到AMD, CommonJS, module.export, import, require 这些东西, 是真的头大.
为什么会有这些东西出现呢, 都是什么玩意?

### 故事起源

在没有这些东西以前, 世界是什么样子的?

JS 中我们最常使用的就是函数, 对象. 用这两的概念, 问题基本能解决的七七八八了.
为了结构清楚, 我们一般会抽出一些 JS 文件, 比如 `dataUtil.js`, `imageUtil.js` 这样的, 来放不同 util 相关的变量和函数.
故事到这里其实还是挺不错的.

直到我们把这些好用的工具(utils)包, 推荐给了其他人.
然后就是无可避免的, 命名冲突.
我们的工具包污染了全局命名, 这些和别人定义的相同的变量名,函数名, 搞垮了别人的库.

对象和函数, 作为别人的依赖, 总是不好直接复用的.
我们需要导出的, 应该是隐藏了内部实现, 提供外部接口的模块.

### 模块模式

众所周知, JS 变量作用域只有全局和函数.
所以要给别人用的包, 为了不污染全局, 不能直接在全局定义变量, 也不能在全局定义函数.

...这, 还写个锤子?

不怕, JS 有立即执行函数, 我们可以定义一个函数并且立即执行, 这个函数执行返回的结果就是我们的接口.
使用的地方只要定义一个变量接收一下 立即执行函数 的返回结果就成了.
返回的结果, 我们可以包在一个对象里面.

比如我们需要一个模块, 模块对外暴露一个函数 `invokeCount`, 这个函数调用后会输出被调用的次数.
这个 count 肯定不能定义在函数里面. 
那它看起来就大概是这样的
```js
(function module(){
  let count = 0
  function invokeCount(){
    console.log(++count)
  }
  
  return {
    invokeCount: invokeCount
  }
})()
```

这个函数会立即执行, 返回的结果就是我们期望的 module 对象.
我们没有污染全局, 所有使用的地方可以把它的返回值定义为任何可用的变量名.
比如我们在 module 中使用的 `count`

```js
const count = (function module(){
  let count = 0
  function invokeCount(){
    console.log(++count)
  }
  
  return {
    invokeCount: invokeCount
  }
})()

count.invokeCount() // 1
count.invokeCount() // 2
count.invokeCount() // 3
```

在 JS 中, 这种通过使用立即执行函数,对象和闭包来创建模块的方式, 就叫做模块模式.
一旦我们能定义模块, 那不同的 util 拆到不同的文件, 就没有任何问题了.

关于闭包的具体内容, 可以看[这里](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)

### 模块扩展

一点小需求, 如果我要输出调用的次数, 以及每次调用的时间, 该怎么办呢?  
option 1 : 重新写一个 module.  
option 2 : 扩展一下上面那个.

这里就不对比了, 我们直接选2.

我们需要的, 就是一个能够接收一个 module, 并且能输出一个函数 `invoke`
这个函数调用后, 即输出被调用的次数, 也输出被调用的时间.

和上面的实现方式基本一致
```js
function superModule(module){
  function invoke(){
    module.invokeCount()
    console.log(new Date())
  }
  
  return {
    invoke: invoke
  }
}
```

使用的方式:
```js
// 简单的 module
const countModule = (function module(){
  let count = 0
  function invokeCount(){
    console.log(++count)
  }
  
  return {
    invokeCount: invokeCount
  }
})()

// 扩展的 super module
const superCount = (function superModule(module){
  function invoke(){
    module.invokeCount()
    console.log(new Date())
  }
  
  return {
    invoke: invoke
  }
})(countModule)
```

模块的扩展是基本操作.
但是和模块模式一样, 都有一些问题.
扩展的模块, 没有办法访问被扩展模块的内部属相.
扩展的模块, 严重依赖模块的顺序. 当你一个业务依赖了很多相互依赖的模块的时候, 就会非常绝望了.

### 解决方案的出现

#### AMD

解决思路1: 基于在**浏览器中的使用**考虑.
我们关注点总共有3个, 被依赖的模块是什么, 模块本身的实现, 以及模块如何被别的模块使用(模块名/ID).
用 superCount 扩展 super 的例子来说就是:
如下定义一个 superCount 模块, 依赖 `count` 模块, 以及自己的实现.

```js
define("superCount", ["count"], (count) => {
  function invoke(){
    count.invokeCount()
    console.log(new Date())
  }
  
  return {
    invoke: invoke
  }
})
```

自动处理依赖, 异步加载模块, 同以文件可以定义多个模块.
以上, 就是基于浏览器使用所定义的 异步模块定义, (Asynchronous Module Definition) AMD.

#### CommonJS

解决思路2: 基于**通用 JS 环境**使用考虑(比如Node.js).
最好还是一个 module 放在一个文件.

文件中像平常一样写自己的 JS 代码, 完全不用考虑全局污染这些问题.
依赖的 module, 使用 require 引入.
只要在最后, 把 return 的对象内容, 写到 module.exports 中就可以了.

引用关键字 `require`, 导出关键字 `module.exports`

```js
const count = require("./countModule.js")

function invoke(){
  count.invokeCount()
  console.log(new Date())
}

module.exports = {
  invoke: invoke
}
```

这样一个文件, 大概会被转换成:
```js
(function (){

  const module = {
    exports: {}
  }
  
  function (){
    const count = modules["countModule"]

    function invoke(){
      count.invokeCount()
      console.log(new Date())
    }

    module.exports = {
      invoke: invoke
    }
  }
  
  return module.exports
})()
```

CommonJS 不显示的支持浏览器, 因为浏览器不支持 module 变量和 export 属性.
我们必须用浏览器支持的打包方式, Bowserify/RequireJS.


### ES Module

取两者所长
AMD: 异步加载模块
CommonJS: 语法简单, 基于文件.

使用 `import xxx from`, `export` 关键字, 加上 ES6 支持的对象结构. 

// superCount.js
```js
import count from "./countModule.js"
// 也可以只引入需要的部分
// import { invokeCount } from "./countModule.js"

function invoke(){
    count.invokeCount()
    // 如果部分引入了 invokeCount, 上面就不用 count.  直接调用就可以
    console.log(new Date())
}

export {
   invoke
}
```

 
导出部分, 也可以不单独写, 直接在定义函数时导出
```js
export function invoke() {...}
```
这样 export, import 的时候就只能使用 
```js
import { invoke } from "./superCount.js
```

ESM 看着挺美好的, 但是直到今天, Node 还不支持, 想要在在项目中使用的话, 还是要靠一些工具.

今天就到这里了.


