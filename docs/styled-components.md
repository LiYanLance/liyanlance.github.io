## Styled Components

### 组件化系统中的最佳实践

-- [来源](https://www.smashingmagazine.com/2017/01/styled-components-enforcing-best-practices-component-based-systems/#comments-styled-components-enforcing-best-practices-component-based-systems)

1. 使用小型、功能单一、独立的组件
2. 分离**逻辑组件**与**展示组件**
3. 使用唯一的 CSS 类名

#### 使用小型、功能单一、独立的组件
不用依赖 classname 来组合，而是利用组件来发挥自己的优势并将它们组合在一起. 
例如, 在 Button 组件内用 props 指定 classname, 而不是在使用 button 的每一个组件里分别指定 className. 这样就不用创建和使用很多的 class.

```js
function Button(props) {
  const className = `btn${props.primary ? ' btn—-primary' : ''}`
  return (
    <button className={className}>{props.children}</button>
  );
}
```
#### 分离逻辑组件与展示组件
将处理数据和逻辑的组件, 与展示样式的组件剥离开. 
如果后端 API 数据发生变化, 只需要改动和处理数据相关的组件. 能确保 UI 不发生变化. 另外, 如果 UI 需要发生变化, 只要找到展示组件进行修改. 这样可以避免同时进行多个不相关组件的更改，从而避免意外 error. 

#### 使用唯一的 CSS 类名
相信大家都经历过被同名 class 支配的恐惧.

###  CSS-in-JS
传统的 web 开发推崇 HTML、CSS、Javascript 都分离.
"CSS-in-JS" 是指一种模式，指 CSS 由 JavaScript 生成而不是在外部文件中定义. 
就像 React 把 HTML 和 JavaScript  放在同一个文件, Style Component 进一步把 CSS 也放入了 JS 中.

Style component 是对 CSS-in-JS 的一个(非常热门的)具体实现, 更多不同的技术方案[在这](https://github.com/MicheleBertoli/css-in-js).
React 对样式如何定义并[没有明确态度](https://reactjs.org/docs/faq-styling.html#what-is-css-in-js). 是否使用完全取决于个人/团队的情况.

#### 为什么要使用 CSS-in-JS
-- [来源](https://mxstbr.com/thoughts/css-in-js/)

* **提升自信, 无痛维护**: 增强写代码的信心: 对一个组件的样式修改永远不会影响另外一个.
* **帮助团队**: 避开常见的CSS困扰，例如类名冲突. 保持代码库整洁
* **更好的性能表现**: 仅将关键的CSS发送给用户，以便快速进行首次绘画
* **动态样式**: 能基于全局主题或者组件状态改变样式

### Styled component 的优势

Styled component 既然是 CSS-in-JS 的实现, 当然会拥有上面的优势, 官网给出的好处(和上面基本对应一致):

* 关联 CSS 和组件: track 哪些组件 render 在了页面上, 注入对应的 style, 用户加载的代码量最少
* 没有 classname 的 bug: 自动生成唯一的 className, 避免冲突/拼写错误
* 可轻松删除 CSS: 每个样式都与特定组件相关, 很容易找到并删除定义了但没有使用的样式.
* 简单动态的样式: 可以基于 state/props 改变样式, 不用维护一堆 classname (通过变量切换)
* 容易维护: 不用去别的文件来查找是哪里影响了当前 component 的样式


**附加作用**: 强迫使用者以组件为最小单位来进行开发. CSS-in-JS


#### 关于避免 classname 冲突
styled component 会给生成的 React 组件添加一个值为随机字符串的 className。使用同一个 styled component 生成的多个 React 组件的 className 是不同的，这种随机 className 的机制使得组件之间的 className 值不会冲突.

## Get start

### 创建一个基于 tag 的 component

不用再去 React component 和 CSS 文件之间找映射关系
创建一个 Title 组件, render 一个 h1 tag. 并加其他点样式. 这就是一个展示组件.
```js
const Title = styled.h1`
  font-size: 1.5em;
  text-align: center;
  color: palevioletred;
`;
```

像普通组件一样使用
```js
return (<Title>这里的 title element 会有上面的样式</Title>)
```

### 基于 props 适配样式

可以将一个 函数（"插值"）传递给 styled component 的字面量，从而根据 props 调整样式

```js
const Button = styled.button`
  background: ${props => props.primary ? "palevioletred" : "white"};
  color: ${props => props.primary ? "white" : "palevioletred"};
`

render(
  <div>
    <Button>Normal</Button>
    <Button primary>Primary</Button>
  </div>
)
```

### 扩展 styled-component 样式

很多时候我们想用一个 component, 但需要改一点点样式. 通常需要传入一个函数, 用 props 改变样式, 这就需要很大 effort.
styled() 构造函数, 可以让我们传入一个 component, 对其增加一些样式, 最后得到一个新的 component.

```js
const Button = styled.button`
  color: palevioletred;
  font-size: 1em;
  border: 2px solid palevioletred;
`;

const TomatoButton = styled(Button)`
  color: tomato;
  border-color: tomato;
`;

render(
  <div>
    <Button>Normal Button</Button>
    <TomatoButton>Tomato Button</TomatoButton>
  </div>
);
```

有的时候, 你可能想把原有组件 render 出的 tag 改掉. 比如 `<button>` 改成 `<a>`.
例如当 build 一个导航栏时, 里面有一堆混合的 a link 和 button, 但是他们的样式需要完全一致.
对于这种情况. 可以使用 `as`. 把 `<button>` render 成 `<a>`

```js
const Button = styled.button`
  color: palevioletred;
  font-size: 1em;
  border: 2px solid palevioletred;
`;

const TomatoButton = styled(Button)`
  color: tomato;
  border-color: tomato;
`;

render(
  <div>
    <Button>Normal Button</Button>
    <Button as="a" href="/">Link with Button styles</Button>
    <TomatoButton as="a" href="/">Link with Tomato Button styles</TomatoButton>
  </div>
)
```

### 扩展任意 component 的样式

你可以对任意组件, 无论自己写的还是第三方的, 只要把 className 传进去.
注意, 与上面不同, 这里的 Component Link 不是 styled-component, 仅仅是普通的 React Component. 
需要在被 styled 的 component 中加 className. 
问题: 感觉不好操作啊, 如果是第三方的组件呢? 只能多包装一层了吗?

```js
const Link = ({ className, children }) => (
  <a className={className}>
    {children}
  </a>
);

const StyledLink = styled(Link)`
  color: palevioletred;
  font-weight: bold;
`;

render(
  <div>
    <Link>Unstyled, boring Link</Link>
    <br />
    <StyledLink>Styled, exciting Link</StyledLink>
  </div>
);
```

### 如何传递 props

如果被 styled 的 target 是一个
* HTML element `styled.div`, 则 HTML 标准属性都会被传到 DOM 上. 
* React component `styled(MyComponent)`, 那么所有 props 都会被传下去.

```js
const Input = styled.input`
  padding: 0.5em;
  margin: 0.5em;
  color: ${props => props.inputColor || "palevioletred"};
  background: papayawhip;
  border: none;
  border-radius: 3px;
`;

render(
  <div>
    <Input defaultValue="@probablyup" type="text" />
    <Input defaultValue="@geelen" type="text" inputColor="rebeccapurple" />
  </div>
);
```

`type` 和 `defaultValue` 被传递给了 DOM, 所以两个 component render 出来都有 type 和 defaultValue. styled-component 可以自动过滤非标准属性.
`inputColor` 不是 HTML 元素的标准属性, 并没有传递给 DOM. 上面的例子中对其进行了处理. 转换成了 DOM 识别的属性 (color)


### 通过 attr 附加 props

为了避免一些没有必要的 wrap, 可以使用 attr 给组件增加一些 props.
```js
const Input = styled.input.attrs(props => ({
  size: props.size || "1em",
}))`
  font-size: 1em;
  border: 2px solid palevioletred;

  /* 动态计算的 props */
  padding: ${props => props.size};
`;

render(
  <div>
    <Input placeholder="A small text input" />
    <Input placeholder="A bigger text input" size="2em" />
  </div>
);

```

## 高级

### 主题

通过 ThemeProvider, render tree 中的所有样式化的组件都可以访问提供的主题.

```js
const Button = styled.button`
  color: ${props => props.theme.main};
  border: 2px solid ${props => props.theme.main};
`

const theme = {
  main: "red"
}

render(
  <ThemeProvider theme={theme}>
    <Button>Themed</Button>
  </ThemeProvider>
)
```
上面 render 出来的 button 就是红的.

theme 的定义还可以是个 function, 入参是外层 themeProvider 的 theme
```js
const darkTheme  = ({red} => {
  main: "grey"
})


render(
  <ThemeProvider theme={theme}>
    <ThemeProvider theme={darkTheme}>
      <Button>Inverted Theme</Button>
    </ThemeProvider>
  </ThemeProvider>
)
```
上面的 button 就是灰的.

同时 Styled-components 还支持以 HOC `withTheme` 或 hooks `useContext(ThemeContext)` 的方式获取 theme.

### Style Objects

Styled-components 支持将 CSS 编写为 JavaScript 对象而不是字符串.
```js
const Box = styled.div({
  background: 'palevioletred',
  height: '50px',
})

const PropsBox = styled.div(props => ({
  background: props.background,
  height: '50px',
}));
```
### 引用其他组件
希望在 hover 到父组件(Link)的时候, 修改某一个子组件(Icon)的样式.

```js
const Link = styled.a`
  padding: 50px 100px;
  color: palevioletred;
`

const Icon = styled.div`
  width: 48px;
  height: 48px;

  ${Link}:hover & {
    background: red;
  }
`

render(
  <Link href="#">
    <Icon />
  </Link>
)
```
## etc.

还有一些和 [服务端渲染](https://www.styled-components.com/docs/advanced#server-side-rendering) 及 [与CSS共存](https://www.styled-components.com/docs/advanced#server-side-rendering) 的解决方案. 包括怎么避免和第三方样式的冲突等等.

以及很多工具: typescript 插件, Jest 集成 ...
就不一一列举了. 
