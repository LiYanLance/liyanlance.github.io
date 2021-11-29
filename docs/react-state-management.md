
# React 应用状态管理

> 这篇文章翻译自 [Kent C. Dodds](https://kentcdodds.com/) 的 [Application State Management with React](https://kentcdodds.com/blog/application-state-management-with-react)

状态管理可以说是所有应用中最难处理的一部分。这也是为什么当下存在这么多的状态管理工具，并且仍然层出不穷（有一些工具甚至建立在另一些之上，npm 中有大量 “简单版的 redux“）。然而，我认为正是由于我们经常过度设计，才导致这个问题这么难处理。

从一开始使用 React，我就在尝试使用一种状态管理方法，而且这个方法还随着 React hooks 的出现（以及 React context 的巨大提升）更加容易被使用了。

我们通常把 React 组件看做乐高积木，用它们来搭建应用。我觉着听到这个说法的人，通常会隐隐地觉得这个说法遗漏了和状态相关的那一部分。我自己使用的方法的“秘密”就是：对待状态管理问题时，想想怎么把应用的状态映射到应用的树状结构上面去。

redux 大获成功的原因之一就是它解决了[Prop Drilling](https://kentcdodds.com/blog/prop-drilling)问题。通过把组件传给一些神奇的 `connect` 函数就可以让数据共享到应用树的任意地方的做法确实很棒。对 reducers/action creator 的使用也不错，但我仍然坚信 redux 的被普遍使用的原因是它为开发者解决了 prop drilling 所带来的痛苦。

我至今只在一个项目中使用过 redux，因为我经常看到开发者把他们所有的状态（state）都放到 redux 中。包括全局状态和本地状态。这会导致非常多的问题，其中最重要的一个是，当你在维护任何状态交互时，都将会涉及到 reducer 、action creator / types 和 dispatch 调用的交互，这最终导致我们必须打开一大堆文件，并在大脑中追溯代码实现，才能弄明白当下发生了什么，以及它对代码库的其它部分产生了什么样的影响。

澄清一下，这样对于全局状态来说是没问题的，但是对于简单的状态（比如一个弹窗是否打开，或者表单中填写的值）来说就会是很大的问题。更糟糕的是，这样基本没法扩展。你的应用越大，这个问题就越难处理。当然，用不一样的 reducer 去管理应用中的不同部分是没问题的，但是通过这些 action creators 和 reducer 来间接处理的方式并不是最好的。

就算没有使用 redux ，把应用中的所有状态全放在一个对象上还是会导致其他问题。当 React `<Context.Provider>` 获取到一个新的值，所有消费它的组件都会被更新且必须被渲染，哪怕它是一个只关心其中部分数据的函数组件。这就会带来潜在的性能问题（React-Redux v6 尝试使用这个办法，然后发现它不能和 hook 一起工作，这导致他们在 v7 中需要用其他办法来处理）。我的重点在于，如果把状态从逻辑上分隔开并且放在 React 树上对应合适的位置，那你就不用担心这些问题了。


* * *

如果你在使用 React 来创建应用，那么其实你已经有了一个状态管理工具。根本不用任何 `npm install` （或 `yarn add`）。你的用户不需要加载更多的数据，它和 npm 上的所有的 React 包集成在一起，并且 React 团队已经对它进行了详细地记录。这个状态管理工具，就是 React 自己。

> React 本身就是一个状态管理工具

当你使用 React 创建应用时，通常是从 `<App />` 组件开始，最后用很多的 `<input />`，`<div />` 和 `<button />` 等等一大堆组件组装出一棵组件树出来。你不会把所有的低阶组件都管理在一处。相反，你会让每个单独的组件管理自己的那一部分，如今用这种方式构建用 UI 是很高效的。状态管理也是如此，你大概率会这么做：

```jsx
function Counter() {
  const [count, setCount] = React.useState(0)   // 状态
  const increment = () => setCount(c => c + 1)
  return <button onClick={increment}>{count}</button>
}

function App() {
  return <Counter />
}
```

这些也可以在类组件中工作，hook 只是让事情做起来更简单一点（尤其是在处理 context 的时候，我们马上就会说到这个）。

```jsx
class Counter extends React.Component {
  state = {count: 0}  // 状态
  increment = () => this.setState(({count}) => ({count: count + 1}))
  render() {
    return <button onClick={this.increment}>{this.state.count}</button>
  }
}
```

”好好好，Kent（作者），在一个组件里维护一个元素当然简单了，但是如果我要在组件之间共享状态该怎么做？比如这样“

```jsx
function CountDisplay() {
  // 这个 `count` 从哪来呢？
  return <div>The current counter count is {count}</div>
}

function App() {
  return (
    <div>
      <CountDisplay />
      <Counter />
    </div>
  )
}
```

”这个 `count` 在 `<Count />` 里面管理着，现在我需要一个状态管理工具帮我在 `<CountDisplay />` 里面访问 `count` ，并且在 `<Counter />` 里更新它！“

这个问题所对应答案的历史和 React 本身一样久远（或者更久远？），而且一直记录在文档中：[状态提升](https://reactjs.org/docs/lifting-state-up.html)。

使用”状态提升“啦解决 React 中的状态管理问题是非常合理的，这一点是不会动摇的。在这个情况下你应该：

```jsx
function Counter({count, onIncrementClick}) {
  return <button onClick={onIncrementClick}>{count}</button>
}

function CountDisplay({count}) {
  return <div>The current counter count is {count}</div>
}

function App() {
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(c => c + 1)
  return (
    <div>
      <CountDisplay count={count} />
      <Counter count={count} onIncrementClick={increment} />
    </div>
  )
}
```

我们简单改变一下负责维护这个状态（count）的组件，我想看起来应该很直观。可以一路把这个状态提升到应用的顶部去。

”不错，Kent，但是你要怎么处理 [prop drilling](https://kentcdodds.com/blog/prop-drilling) 问题“

好问题，我们首先可以改变构建组件的方式，这样就有了 [component composition](https://reactjs.org/docs/context.html#before-you-use-context) 的优势。与其写成这样，

```jsx
function App() {
  const [someState, setSomeState] = React.useState('some state')
  return (
    <>
      <Header someState={someState} onStateChange={setSomeState} />
      <LeftNav someState={someState} onStateChange={setSomeState} />
      <MainContent someState={someState} onStateChange={setSomeState} />
    </>
  )
}
```

你更应该这样做：

```jsx
function App() {
  const [someState, setSomeState] = React.useState('some state')
  return (
    <>
      <Header
        logo={<Logo someState={someState} />}
        settings={<Settings onStateChange={setSomeState} />}
      />
      <LeftNav>
        <SomeLink someState={someState} />
        <SomeOtherLink someState={someState} />
        <Etc someState={someState} />
      </LeftNav>
      <MainContent>
        <SomeSensibleComponent someState={someState} />
        <AndSoOn someState={someState} />
      </MainContent>
    </>
  )
}
```

如果这里不是特别清楚（这个例子稍微有些刻意）， [Michael Jackson](https://twitter.com/mjackson) 有一个非常棒的 [视频](https://www.youtube.com/watch?v=3XaXKiXtNjw) 可以帮你理解我想说什么。

到最后，组件组合也没办法解决问题的时候，你还是走向了 React Context API。这一直以来是一个”解决方案“，但是同时也是”非官方“的解决方案。 就像我说的，很多人用 `react-redux` 是因为它使用了我提到的机制，同时也不同去担心 React 文档中的警告。 但是如今 `context` 已经被官方正是支持，我们直接使用是没有任何问题的：

```jsx
// src/count/count-context.js
import * as React from 'react'

const CountContext = React.createContext()

function useCount() {
  const context = React.useContext(CountContext)
  if (!context) {
    throw new Error(`useCount must be used within a CountProvider`)
  }
  return context
}

function CountProvider(props) {
  const [count, setCount] = React.useState(0)
  const value = React.useMemo(() => [count, setCount], [count])
  return <CountContext.Provider value={value} {...props} />
}

export {CountProvider, useCount}
```

```jsx
// src/count/page.js
import * as React from 'react'
import {CountProvider, useCount} from './count-context'

function Counter() {
  const [count, setCount] = useCount()
  const increment = () => setCount(c => c + 1)
  return <button onClick={increment}>{count}</button>
}

function CountDisplay() {
  const [count] = useCount()
  return <div>The current counter count is {count}</div>
}

function CountPage() {
  return (
    <div>
      <CountProvider>
        <CountDisplay />
        <Counter />
      </CountProvider>
    </div>
  )
}
```

> 注意：这段代码非常非常刻意，我并不推荐你使用 context 解决这种特定场景的问题。 请阅读 [Prop Drilling](https://kentcdodds.com/blog/prop-drilling) 来了解为什么说 Prop Drilling 不是一个问题，反而大多情形下是可取的。不要一开始就使用 context。

这个方法的好处在于我们可以把所有常用来更新状态的逻辑都放在 `useCount` hook 中。

```js
function useCount() {
  const context = React.useContext(CountContext)
  if (!context) {
    throw new Error(`useCount must be used within a CountProvider`)
  }
  const [count, setCount] = context

  const increment = () => setCount(c => c + 1)
  return {
    count,
    setCount,
    increment,
  }
}
```

你也可以很容易地把 `useState` 使用 `useReducer` 替换。

```js
function countReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT': {
      return {count: state.count + 1}
    }
    default: {
      throw new Error(`Unsupported action type: ${action.type}`)
    }
  }
}

function CountProvider(props) {
  const [state, dispatch] = React.useReducer(countReducer, {count: 0})
  const value = React.useMemo(() => [state, dispatch], [state])
  return <CountContext.Provider value={value} {...props} />
}

function useCount() {
  const context = React.useContext(CountContext)
  if (!context) {
    throw new Error(`useCount must be used within a CountProvider`)
  }
  const [state, dispatch] = context

  const increment = () => dispatch({type: 'INCREMENT'})
  return {
    state,
    dispatch,
    increment,
  }
}
```


这给你巨大的自由度，指数级降低了复杂性。这么做有几个要注意的点：
1. 不是应用中所有的状态都需要放在同一个状态对象中。保持它们从逻辑上分离。（用户的设置不需要和提示信息放在一个 context 里面。）你可以用这个方法创建多个 providers。
2. 不是所有的 context 都需要能够被全局访问！尽可能地把状态和需要它的地方放的近一些。


接着上面的第二点，你的应用树可能看起来或许会像是这样：

```jsx
function App() {
  return (
    <ThemeProvider>
      <AuthenticationProvider>
        <Router>
          <Home path="/" />
          <About path="/about" />
          <UserPage path="/:userId" />
          <UserSettings path="/settings" />
          <Notifications path="/notifications" />
        </Router>
      </AuthenticationProvider>
    </ThemeProvider>
  )
}

function Notifications() {
  return (
    <NotificationsProvider>
      <NotificationsTab />
      <NotificationsTypeList />
      <NotificationsList />
    </NotificationsProvider>
  )
}

function UserPage({username}) {
  return (
    <UserProvider username={username}>
      <UserInfo />
      <UserNav />
      <UserActivity />
    </UserProvider>
  )
}

function UserSettings() {
  // 这个可以是和 AuthenticationProvider 有关的 hook
  const {user} = useAuthenticatedUser()
}
```

注意每个页面都可以有自己的 provider，为下层的组件提供必要的数据。 Code Splitting 在这里非常适用。放入每个 provider 的数据，依赖于 provider 使用的 hook 和你在应用中获取数据的方式，但是你知道从哪（ Provider ）开始看代码就能搞懂这一切。

想了解为什么 colocation 是有益的, 可以阅读我的这些文章 [状态 Colocation 可以让你的 React 应用更快](https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster) and [Colocation](https://kentcdodds.com/blog/colocation)。 想了解更多和 context 相关的内容，可以看这篇[如何高效使用 React Context](https://kentcdodds.com/blog/how-to-use-react-context-effectively)

# 服务器缓存和 UI 状态

最后再提一件事。我们有很多种不同类型的状态，都可以归在以下两类中：
1. 服务器缓存 - 存储在服务器上的状态，我们把它存放在客户端是为了快速访问（像是用户数据）
2. UI 状态 - 只在 UI 中有用的状态， 用来控制应用中的交互部分（比如弹窗 `isOpen` 状态）

当我们把两个东西弄混就会出问题。服务器端的缓存和 UI 状态有本质的差异，需要被分开管理。如果你信奉 -- 你拥有的不是正真的状态而是状态的缓存，那么你就可以开始正确地思考它，从而正确地管理它。

你当然可以通过写一些 `useState` 或 `useReducer` 结合 `useContext` 来管理它们。但是让我开门见山地告诉你，缓存是很难解决的问题（有些人说它们计算机科学最难的问题之一），处理这个问题时站在巨人的肩膀上是明智的。

这也是为什么我使用并推荐 [react-query](https://github.com/tannerlinsley/react-query) 来处理这类状态。我知道我说了你不需要其他的状态管理工具，但是我真的不认为它是一个状态管理工具。我把它看做一份缓存。而且是非常TM好的一份缓存。[Tanner Linsley](https://twitter.com/tannerlinsley)(react-query 的作者) 真的牛的不行，了解一下这个工具吧。


# 性能

如果你遵循上面的建议，性能基本不是什么问题。尤其是当你同时也遵循了[关于 colocation 的建议](https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster)。然而，当然会存在性能会出问题的场景。当你有状态相关的性能问题时，首先要做的是检查有多少组件因为一个状态改变需要重新渲染，然后判断这个状态改变的时候它们是不是真的需要被重新渲染。如果需要，那性能问题不是来自于状态管理，而是渲染的速度，你可能会想想了解[为你的 render 方法提速](https://kentcdodds.com/blog/fix-the-slow-render-before-you-fix-the-re-render)。

但是，如果你注意到大量的组件渲染都不会涉及 DOM 更新和所需的副作用，那么这些组件可能在进行无意义的渲染。这在 React 中很常见，它本身通常不是问题。（你需要关注的是[怎么让不必要的重复渲染更快一些]( focus on making even unnecessary re-renders fast first)），如果它真的是瓶颈，这里有几个解决的办法：

1. 把你的状态分为多个逻辑块而不是放在一个 store 里，这样一个状态变化不会导致整个应用中的组件都被更新。
2. [优化你的 context provider](https://kentcdodds.com/blog/how-to-use-react-context-effectively)
3. 引入 [jotai](https://github.com/pmndrs/jotai)

是的，我又推荐了一个状态管理工具。确实会有一些场景，React 自带的状态管理抽象不是特别适用。在所有可用的抽象中，jotai 是最有前途的。如果你对 jotai 的使用场景和能解决的问题有兴趣，这里详细描述了这些内容 [Recoil: State Management for Today's React - Dave McCabe aka @mcc_abe at @ReactEurope 2020](https://www.youtube.com/watch?v=_ISAA_Jt9kI)。Recoil 和 jotai 非常相似（它们解决的是相同类型的问题）。 但根据我对他们的（有限的）经验，我更喜欢 jotai。

在任何情况下，大多数应用程序都不需要像 recoil 或 jotai 这样的原子状态管理工具。


# 结论
同样，这些都可以在类组件中使用（不必非要使用 hook ）。 hooks 使操作更容易，但是你也可以在 React 15 里践行这些思想。 尽可能的保持状态本地化，只有在 prop drilling 真的变成问题时才使用 context。这样可以让你更容易维护状态交互。



