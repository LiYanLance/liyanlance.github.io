
# Jest Snapshot

## 这是什么

一个典型的 snapshot 就是把 UI component render 的结果以代码的形式存储下来, 如果 component 改动影响到了 render 的结构, snapshot 就会发生改变, 导致当前生成的 snaphshot 和之前存储的不一致, 导致测试 failed. 此时我们需要确认引起的 UI 变化是否是我们所期望的, 从而更新 snapshot 或者返回修改代码.


e.g.
对于 Button 组件
```js
<button>
  <img src="some url"/>
  {children}
</button>
```
如果 render `<Button>Click me</button>` 会生成类似的 snapshot, 以 string 的形式保存在一个新创建的文件中, 在当前测试目录的 `./__snapshots__`
```snap
<button>
  <img
    src="some url"
  >
  Click me
</button>
```

## 作用
当我们的改动/重构可能会影响到 UI 的时候, snapshot 能帮助我们确认 UI 是否被影响, 哪些地方被影响了.

e.g. 
对于上面那个 Button component, UT 可以测试的点, img 和 children 存在, 以及更为具体的 props.

但是, 如果改变 img 和 children 的位置, 上面的 UT 就不会出错. 我们 render 的东西改变了, 但是测试却没有 failed.
```js
<button>
  {children}
  <img src="some url"/>
</button>
```
这种情况下, Snapshot 比对就会出错, 提醒 UI 被改动.



## 如何使用 Snapshot

对于 React Element: Button

```js
const Button = ({children}) => (
  <button>
    <img src="some url">
    {children}
  <button>
)
```

需要
1. 把 component render 到内存中, 最终传到 jest 的 matchSnapshot (react-test-render / enzyme)
2. 有一个 serializer 把 snapshot 序列化/stringify, 从而让结果看起来像 DOM.

### 使用 react-test-render
```js
import React from 'react';
import Link from '../Button';
import renderer from 'react-test-renderer';

const component = renderer.create(
  <Button>Click me</Button>
)
const result = component.toJson()
expect(result).toMatchSnapshot()
```

### 使用 Enzyme
```js
import React from 'react';
import Link from '../Button';
import { shallow } from 'enzyme';
import toJson from 'enzyme-to-json'

const wrapper = shallow(<Button/>)
const result = toJson(wrapper)
expect(result).toMatchSnapshot()
```

**serializer** 可以配置到 jest config 中, 不用在每个测试文件中都引入并手动调用 `toJson`.

```json
"snapshotSerializers": ["enzyme-to-json/serializer"]
```
```js
const wrapper = shallow(<Button/>)
expect(wrapper).toMatchSnapshot()
```

另外, 还有直接在测试文件中输出 snapshot 的方法 `.toMatchInlineSnapshot()` 可以查阅[文档](https://jestjs.io/docs/en/snapshot-testing#inline-snapshots)

## 定制 snapshot
从 object 到 string 的过程是由 snapshotSerializers 完成的, 如果对 snapshot 的输出结果(格式)有要求, 可以做定制化, 也可以直接自己写 serializer.

### extend
扩展 expect 方法, 增加自定义的 snapshot 方法.

e.g. 不想在 snapshot 中显示 className, 可以在调用 jest 的 snapshot 前, 删除 className, 再用这个结果 match snapshot.

```js
const {toMatchSnapshot} = require("docs/tests/jest-snapshot")

expect.extend({
    toMatchNonClassSnapshot(received) {
        delete received.props.className
        return toMatchSnapshot.call(this, received, 'toMatchNonClassSnapshot')
    }
})

expect(shallow(<SomeComponent/>)).toMatchNonClassSnapshot()
```

### serializer
自定义 serializer 需要两个参数
- test: 当前节点是否要在 snapshot 中出现.
- print: 当前节点要从 object 到什么样子的 string
```js
expect.addSnapshotSerializer({
  test: (value) => boolean,
  print: (value) => string
})
```

e.g. 对于 enzyme render 的 component
```js
expect.addSnapshotSerializer({
  test(value) {
    return value && value.getElement()
  },
  print(value) {
    // Enzyme Wrapper Element
    const node = value.getElement()
    const props = Object.keys(node.props).filter(key => key !== "children").map(key => {
      let value = node.props[key]
      if(typeof value === "function"){
        value = "[[Function]]"
      }
      return `${key}=${value}`
    }).join(",\n  ")

    const result =
`<${node.type}
  ${props}
>
  ${node.props.children}
</${node.type}>
`
    return result
  }
})
```

### styled-components
正在使用 styled-Components?
可以尝试使用 **jest-styled-components** 让 snapshot 的结果更加友善/定制化

## Good stuff
1. Better coverage - 不用为了测一个 button 写一堆测试
2. Easy to write - 只需 `expect.toMatchSnapshot()`
3. Easy to update - `jest -u`
4. Greate for refactoring - UI 变动的保证

## Bad things
1. No TDD
2. Merge conflicts - snapshot 是 string, git rebase 的时候很难 resolve conflicts
3. Test description - snapshot 增多导致测试作为文档的作用下降
4. Over-use - 因为简单, 容易写太多的snapshot


## Best Practices

1. Treat snapshots as code

提交 snapshot, review snapshot, 把它当做代码的一部分
确保 snapshot 明确，简短. 使用强制执行这些约定样式的工具，以确保 snapshot 可读.

- eslint-plugin-jest (option: no-large-snapshots)
`"rules": { "jest/no-large-snapshots": ["warn", { "maxSize": 12 }] }`
- snapshot-diff: 适用于测试不同 React 组件状态之间的差异.
`expect(snapshotDiff(a, b)).toMatchSnapshot()`

最终的目的是让 PR 中的 snapshot 简单到可 review,
并在出错的第一时间去检查失败的原因, 而不是直接 update snapshot.

2. Snapshot 应该是确定的

不应该每次 run 都不一样, e.g. 组件渲染了 `Date.now()`
这时候应该 mock Data.now 方法, 返回一个确定的值


3. 使用有意义的 snapshot 名字

```js
exports[`<UserName /> should handle some test case`] = `null`
exports[`<UserName /> should render null`] = `null`
```
Tip: 如果一个 test case 中有多个 snapshot 测试, 可以使用 `toMatchSnapshot("some name")` 为每个 snapshot 传入更为具体的 name.

## 相关问题
#### 在测试金字塔中的位置
算 unit 级别的 test, 但是又和典型的 unit test 不同.
```
       UI
    Integration
   UT | Snapshot
```

#### 如何 TDD
通常情况下, TDD 需要先写 test 再完成 code.
Snapshot 反了过来 先 code, 随后 make snapshot, 在后面的每次 test 中检查是否有变化.

Snapshot 致力于帮助确定测试模块的输出是否变化了，而不是在最开始就指导代码的设计.
虽然可以手写 snapshot, 但是它确实不适用于TDD

### visual regression
visual regression 致力于 image 上 pixel 级别的比对
Snapshot 的数据时序列化后存储在文本文件中的, 用字符串 diff


## 资料
### 完整的测试代码
[Snapshot & extend](https://github.com/LiYanLance/hello-jest/blob/master/test/matchers/10_snapshot.test.js)  
[Snapshot with react & serializer](https://github.com/LiYanLance/hello-jest/tree/master/test/with_react)


### 参考资料
https://jestjs.io/docs/en/snapshot-testing#snapshot-testing-with-jest  
https://jestjs.io/docs/en/expect#custom-snapshot-matchers  
https://jestjs.io/docs/en/expect#expectaddsnapshotserializerserializer  
https://www.youtube.com/watch?v=sCbGfi40IWk  
https://www.youtube.com/watch?v=HAuXJVI_bUs  
https://jestjs.io/blog/2016/07/27/jest-14.html#why-snapshot-testing  
