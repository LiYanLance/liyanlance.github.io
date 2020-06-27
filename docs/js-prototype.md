刚刚从 Java 到 Javascript 的时候, 非常不适应 JS 的面向对象, ES6 的 class 关键字也一度让我非常迷惑. 
到底怎么区分 Java 和 JavaScript 在面向对象上的不同呢.
为什么会有人说 JS 是基于对象(或基于原型)的, 不是面向对象的语言.

## 对象

### 基于对象和面向对象
如果一种语言中有面向对象的所有特性, 那就它是面向对象的语言.
如果一种语言中没有, 或者没有完全包含面向对象的特性, 那就是基于对象的语言.

面向对象的基本概念包括: 封装, 继承, 多态.

或许不是很准确, 根据我的理解大体是这样的. 

虽然 JavaScript 一直在快速发展, 但是到今天还是没有完全拥有这些特性. JS 在 ES6 中开始使用 extend 来进行继承, 这也是我最初接触时区别 JS 类型时比较迷惑的一点. 

另外还有一种说法, 将 **基于对象**的编程语言再细分为两类
1. 只实现了部分 OOP 特性的编程语言.
2. 基于 prototype 的编程语言

就上面的说法, JavaScript 是基于对象的. 非要细分, 就是基于 prototype 的.

另外还有 class-based / prototype-based 这种说法, 所以我觉得, 不过刻意在意这些说法. 知道它们是不同的思想就可以了.

### 对象
大多数时候, 不论 JavaScript 还是 Java, 我们的程序都是在操作着一系列对象, 对象之间相互通信, 不断改变状态, 最终达到我们需要的结果.

要想区分基于原型和面向对象, 首先要弄清楚, 是什么对象?

根据 [MDN 上的解释](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Objects/Basics):
对象是一个包含相关数据和方法的集合（通常由一些变量和函数组成，我们称之为对象里面的属性和方法）

一般来说一个对象应该有如下特性:
1. 唯一性标识: 完全相同的两个对象, 也不是同一个对象
2. 有状态: 对象可能会处于不同的状态下
3. 有行为: 对象的状态会因为行为发生改变

大多语言都是用内存地址来区分对象的唯一性标识的.

关于状态和行为 :
```js
const cat = {
    age: 1,
    say: function (){ console.log("miao") }
}
```
对于上面的 cat 对象, `age` 是状态, `say` 是行为.
只不过对于状态和行为, JavaScript 没有作区分, 统一当做属性对待. 不存在 Java 中 "属性" 和 "方法" 的叫法. 当然叫`方法`更便于我们自己区分.

这也涉及 JavaScript 中的另一个概念, 函数是一等公民, 即函数和值一样, 可以被赋给变量. 尽管看起来很奇怪, 但其实 age 和 say 并没有什么不同.

我们可以看到, 没有类, 我们也能创建出一个对象来(上例中的cat), 也就是对象并不一定非要依赖于类来创建. 面向对象的思想一度非常流行, 但是程序运行时, 处理的还是对象. 类并不是创建一个对象所必须的.


### 基于原型

那原型到底是什么?

当你写生时, 景物就是原型.
当你临摹时, 字帖就是原型.

原型, 就是一个样例. 
我们可以把所有的白猫, 黑猫, 花猫都定义为猫这一类别的不同实例.
当然也可以把白猫, 黑猫, 花猫都定位为以白猫为样例的不同对象.

归类 vs. 近似.
就是面向对象和基于原型的差别.

我们既然可以通过初始化一个类来创建对象, 同样也可以通过复制一个原型来创建对象.

基于原型的 JavaScript 正是通过复制的方式, 来创建新的对象的. 当然 JavaScript 中并不是真的去复制一个原型对象, 而是使新对象上有原型的引用.

Java 中的对象必然对应一个 class, JavaScript 中的对象必然对应一个 prototype

## JavaScript 的原型

- 所有对象都有一个私有字段 prototype, 就是这个对象的原型.
- 读一个属性, 如果对象本身没有,就继续访问对象的原型, 直到找到或其原型为空.

ES6 中提供了一些方法可以让我们通过原型来创建新的对象
Object.create 可以通过原型创建一个对象
Object.getPrototypeOf 获得对象的原型
Object.setPrototypeOf 设置对象的原型

### 抽象和复用

```js
const cat = {
    skinColor: "white",
    say: () => { console.log("miao") }
}

// 以 cat 为原型, 创建 blackCat
const blackCat = Object.create(cat, {
    skinColor: {
        value: "black"
    }
})

// cat.isPrototypeOf(blackCat)  true

const whiteCat = Object.create(cat)
console.log(whiteCat.skinColor)

// 通过 blackCat 创建另一个 blkcat
const blkcat = Object.create(blackCat)
console.log(blkcat.skinColor)
blkcat.say()
```

我们先创建了一个 cat 对象, 随后根据 cat 对象创建了 blackCat 对象. 之后我们可以用 Object.create 来复制 cat 或 blackCat 以创建不同的 cat 和 blackCat 对象.

### 早期原型

#### 为什么可以 new 对象

在 ES6 之前, Object 上是没有 create, getPrototypeOf, setPrototypeOf 方法的.
而且我们确实能使用 new 关键字来创建一个对象, 还能使用 this.field 获取对象的属性. 这不就是面向对象的语法吗?

据说这些的出现是因为 JavaScript 创建过程中遭遇了一些政治原因(Java过于流行, 被上层施压), 不是作者的本意. 我们来看看像是 `new Cat()` 这样的语法究竟是怎么回事.

#### 构造器函数

如果我们只看 `Cat()` 本身, 那它和调用一个普通方法没有任何区别, 非要说有, 就是这里函数的首字母是大写的. 这使得 `new Cat()` 看起来更像是在初始化一个类了, 和 Java 没有区别.

但也仅仅是看起来. 大写首字母并不是必须的, 一般约定对于要通过 new 来调用以创建对象的函数, 通常称为构造器函数, 我们会把函数首字母大写. 用于和普通函数区分.

通过构造器函数创建对象并不是 JavaScript 所必须的, 甚至有点多余.

在了解构造器函数怎么工作前, 我们需要有另外一个概念.

##### 函数对象

在 JavaScript 的世界里, function 也是对象. 
同样拥有 prototype 属性.

因为函数是对象, 所以可以被用来赋值给变量, 或者保存在对象和数组中. 可以像使用一个普通的对象一样使用. 当然也可以被当做参数传递或者返回. 作为对象, 函数中也能有函数.

它和普通对象的区别就是可以被调用.

#### new 对象

因为所有的 function 都有可能被当做对象构造器使用. 所以, 所有函数的 prototype 指向的对象上都有 constructor 属性, 这个属性指向当前函数.

比如
```js
function Cat(age) {
    this.age = age;
}
```
如果定义这样一个 Cat 函数, 它的 prototype 就是下面这样的对象.
```js
{ 
    constructor: function Cat(age) {
        this.age = age; 
    }
}
```
注意，constructor 的值是对函数本身的引用，不是字符串.
你可以一路 `Cat.prototype.constructor.prototype.constructor` 循环调用下去...

这也能回答为什么我们在判断一个对象类型的时候用的是
```js
cat.constructor === Cat
```
但是, 一旦 prototype 被修改, 这个方法也就失效了, 例如
```js
function Cat(age) { this.age = age; }

Cat.prototype = "dummy"
const cat = new Cat()
cat.consturctor === Cat // false
```
因为原型上的 prototype 已经变化了. 

当使用 new 关键字来调用的时候, 其实被构造出来的是函数上 prototype 属性中 consturctor的新对象. 同时把 this 指针指向这个对象.

#### new 如果是 function
如果 new 关键字是一个方法, 那它的实现或许是这样:
```js
function new(){
  const that = Object.create(this.prototype) // 基于原型创建对象
  const other = this.apply(that, argument) // 调用对象的 constructor
  return (typeof other === "object" && other) || that
}
```
如果 `that` 对象的构造函数返回的 `other` 是一个对象, 就返回 `other`, 不然就说明构造函数只进行了初始化, 那最后就返回我们基于原型创建的对象 `that`

对于构造器函数 Cat
```js
function Cat(name, age){
  this.name = name
  this.age = age
}
```
使用构造器创建对象
```js
const blackCat = new Cat("bk", 1)
``` 
就相当于
```js
const cat = Object.create(Cat.prototype) // 创建一个 cat {} 空对象
Cat.prototype.constructor.apply(cat, ["bk", 1]) // 初始化空对象, Cat("bk", 1) 没有返回值
const blackCat = cat
```

### 高度动态

JavaScript 的对象比 Java 对象设计灵活的地方是 其具有高度的动态性. 可以在运行时向对象内添加属性.
 
 对于 `Cat` 
```js
function Cat(name, age){
  this.name = name
  this.age = age
}
```
可以在创建之后对其添加新的独有属性/方法.
```js
const cat = new Cat("cici", 2)
cat.jump = () => { console.log("jump") }
```

也可以直接向其原型添加属性/方法, 使所有以其为原型的所有对象, 能通过原型访问这个添加的属性.
```js
Cat.prototype.jump = function(){
  console.log(this.name + "jump")
}
```

### 区分原型属性/方法

大多对象的 prototype 最终都指向了 Object.prototype, 当我们在对象上调用一个方法时, 首先在当前对象找, 然后会顺着原型链一路向上找.

比如,对
```js
cat.toString()
```
cat 对象上没有 toString 方法, 其原型 Cat.prototype 上也没有, 最终会调用 Object.toString

如何区分哪些属性/方法是当前对象的呢?

可以使用 `hasOwnProperty` 来判断某一个属性/方法
```js
cat.hasOwnProperty("name")
```

#### for in
也可以使用 for in 来枚举所有非原型链中的属性
```js
for (name in cat){
   // 所有属性
  if( typeof cat[name] !== "function"){
    console.log(name)
  }
}
```
