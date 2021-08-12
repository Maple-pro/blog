---
title: unstated-next 实现组件间数据逻辑共享
date: 2021-08-12 20:59:14
tags: [React, Hooks, unstated-next]
categories: [学习笔记, React]
description: 介绍 unstated-next 这个 React 状态管理库，Context -> useContext -> unstated-next 这个过程中的演变
---

> https://github.com/jamiebuilds/unstated-next

# API 使用

https://github.com/jamiebuilds/unstated-next/blob/master/README-zh-cn.md#api

`createContainer(useHook)`
`<Container.Provider>`
`<Container.Provider initialState>`
`Container.useContainer()`
`useContainer(Container)`

```typescript
// @/pages/index/connect
import Main from '@/pages/index/hooks';

// class 组件使用状态
export const connect = fn => Component => props => {
  const state = Main.useContainer();
  const watchState = fn(state);
  return <Component {...watchState} {...props}></Component>;
};

// function 组件使用状态
export const useStore = () => {
  return Main.useContainer();
};
```

```typescript
// @/pages/index/index.jsx
import { connect, useStore } from '@/pages/index/connect';

@connect(state => ({
  taskList: state.taskList,
  triggerTask: state.triggerTask,
}))
class A extends Component {
  constructor(props) {
    super(props);
  }
  
  render() {
    const { taskList, triggerTask } = this.props;
    return ();
  }
}

function B() {
  const { reload } = useStore();
}
```

组件 A 中的 `props` 由两部分构成：

- `connect` 中 `fn(state)` 返回的值
- 调用组件时传入的 `props`

# 详细解析

## React 中 Context 的使用

https://zh-hans.reactjs.org/docs/context.html

官网的链接已经说得很详细了，这里只做简单的摘录

### 什么时候使用？

为了共享一些对于一个组件树而言是全局的数据，例如当前认证的用户、首选语言等，可以避免 props 的层层传递。

### 组件组合

使用组件组合的方式避免 props 的层层传递，但这种会使得高层组件变得复杂。

参考 [组合 🆚 继承](https://zh-hans.reactjs.org/docs/composition-vs-inheritance.html#containment)

尽量使用组合而非继承。

### API

`React.createContext()`
`Context.Provider`
`Class.contextType`
`Context.Consumer`
`Context.displayName`

## React 中 useContext 的使用

```typescript
const value = useContext(MyContext);
```

接收一个 context 对象，并返回该 context 对象当前的值。当前的 context 值由上层组件中距离当前组件最近的 `<MyContext.Provider>` 的 `value` props 决定。

相当于 `static contextType = MyContext` 或 `<MyContext.Consumer>`

仍然需要和 `<MyContext.Provider>` 配合使用

## unstated-next

### 为什么使用？

将 `setState` 从具体的 UI 组件中剥离，形成一个数据对象实体，可以被其他 UI 组件复用。

- 使用 Hooks，只能共享逻辑，但无法共享 state。自定义 Hook 是一种重用状态逻辑的机制(例如设置为订阅并存储当前值)，所以每次使用自定义 Hook 时，其中的所有 state 和副作用都是完全隔离的。
- 使用 context，可以共享 state，但代码结构复杂。

```typescript
// 源自官网代码示例
function useCounter() {
  let [count, setCount] = useState(0)
  let decrement = () => setCount(count - 1)
  let increment = () => setCount(count + 1)
  return { count, decrement, increment }
}

let Counter = createContext(null)

function CounterDisplay() {
  let counter = useContext(Counter)
  return (
    <div>
      <button onClick={counter.decrement}>-</button>
      <p>You clicked {counter.count} times</p>
      <button onClick={counter.increment}>+</button>
    </div>
  )
}

function App() {
  let counter = useCounter()
  return (
    <Counter.Provider value={counter}>
      <CounterDisplay />
      <CounterDisplay />
    </Counter.Provider>
  )
}
```

### 代码解读

From https://github.com/jamiebuilds/unstated-next/blob/master/src/unstated-next.tsx

```typescript
import React from "react"

const EMPTY: unique symbol = Symbol()

export interface ContainerProviderProps<State = void> {
	initialState?: State
	children: React.ReactNode
}

export interface Container<Value, State = void> {
	Provider: React.ComponentType<ContainerProviderProps<State>>
	useContainer: () => Value
}

export function createContainer<Value, State = void>(
	useHook: (initialState?: State) => Value,
): Container<Value, State> {
	let Context = React.createContext<Value | typeof EMPTY>(EMPTY)

	function Provider(props: ContainerProviderProps<State>) {
		let value = useHook(props.initialState)
		return <Context.Provider value={value}>{props.children}</Context.Provider>
	}

	function useContainer(): Value {
		let value = React.useContext(Context)
		if (value === EMPTY) {
			throw new Error("Component must be wrapped with <Container.Provider>")
		}
		return value
	}

	return { Provider, useContainer }
}

export function useContainer<Value, State = void>(
	container: Container<Value, State>,
): Value {
	return container.useContainer()
}
```

内容：使用 `createContainer` 将自定义 Hooks 封装成一个数据对象，提供 `Provider` 注入和 `useContainer()` 获取 store 这两个方法。

# 代码结构

1. 写 service  `common/service/index.js` 调用 bff 层的 rpc 接口
2. hook 层存方法 `pages/index/hooks/*.js`

3. connect 导出，`common/connect`
4. 组件中使用 `connect` 或 `useStore` 使用 react Hooks 中的数据和方法