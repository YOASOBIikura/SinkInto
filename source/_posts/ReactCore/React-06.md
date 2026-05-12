---
title: React-06
date: 2026-05-12 13:32:12
categories:
  - ReactCore
tags:
  - Hooks / useState
---

# 第 6 讲：Hooks / useState

核心文件：

- `packages/react/index.ts`
- `packages/reconciler/FiberHook.ts`
- `packages/reconciler/BeginWork.ts`
- `packages/reconciler/WorkLoop.ts`

这一讲要回答的问题是：

> 函数组件里的 `useState` 是怎么保存状态，并触发重新渲染的？

前面我们已经知道函数组件会在 `beginWork` 中被执行：

```ts
case FunctionComponent:
  const children = renderWihtHooks(fiber, fiber.type);
  fiber.child = reconcileChildFibers(fiber, children)
  return fiber.child;
```

这里的关键不是直接调用：

```ts
fiber.type()
```

而是调用：

```ts
renderWihtHooks(fiber, fiber.type)
```

因为函数组件可能会使用 hooks。

---

## 1. `useState` 从哪里导出

核心文件：

```txt
packages/react/index.ts
```

源码：

```ts
export const ReactSharedInternals: any = {
  H: null
}

export function useState(initialState: any) {
  return ReactSharedInternals.H(initialState);
}
```

表面上看，`useState` 只是调用：

```ts
ReactSharedInternals.H(initialState)
```

这里的 `H` 可以理解成当前 hooks dispatcher。

也就是说：

> 真正执行 `useState` 逻辑的函数，不在 `react/index.ts` 里，而是在 reconciler 渲染函数组件前临时设置进去。

---

## 2. 为什么要有 dispatcher

函数组件可能处于不同阶段：

- 首次渲染，也就是 mount 阶段。
- 状态更新后的再次渲染，也就是 update 阶段。

这两个阶段 `useState` 的行为不一样。

mount 阶段：

```txt
创建 hook
保存初始状态
创建 dispatch
挂到 Fiber.memoizedState 上
```

update 阶段：

```txt
找到已有 hook
读取已有状态
返回之前保存的 dispatch
```

所以同一个：

```ts
useState(0)
```

在不同阶段实际要调用不同函数：

```txt
mount 阶段  -> mountState
update 阶段 -> updateState
```

这就是 `ReactSharedInternals.H` 的作用。

---

## 3. Hook 数据结构

源码：

```ts
export type Hook = {
  memoizedState: any,
  dispatch: any,
  next: Hook | null,
}
```

一个 hook 保存三样东西：

```txt
memoizedState -> 当前状态值
dispatch      -> 更新状态的方法
next          -> 下一个 hook
```

比如：

```tsx
function App() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState("du1");
}
```

会形成 hook 链表：

```txt
hook(count)
  memoizedState: 0
  dispatch: setCount
  next -> hook(name)

hook(name)
  memoizedState: "du1"
  dispatch: setName
  next -> null
```

这个 hook 链表挂在函数组件 Fiber 上：

```txt
App Fiber.memoizedState -> 第一个 hook
```

---

## 4. 两个全局变量

源码：

```ts
let currentlyRenderingFiber: Fiber | null = null;
let workInProgressHook: Hook | null = null;
```

`currentlyRenderingFiber` 表示：

> 当前正在渲染的函数组件 Fiber。

比如执行 `App()` 时，它就是 `App Fiber`。

`workInProgressHook` 表示：

> 当前正在处理的 hook。

因为一个函数组件里可以有多个 `useState`，所以需要用它沿着 hook 链表往后走。

---

## 5. `renderWihtHooks`

源码：

```ts
export function renderWihtHooks(workInProgress: Fiber, Component: any) {
  currentlyRenderingFiber = workInProgress;

  if (currentlyRenderingFiber!.memoizedState === null) {
    ReactSharedInternals.H = mountState;
  } else {
    ReactSharedInternals.H = updateState;
  }

  const result = Component();
  workInProgressHook = null;
  currentlyRenderingFiber = null;
  return result;
}
```

这个函数做了几件事。

第一，记录当前正在渲染的 Fiber：

```ts
currentlyRenderingFiber = workInProgress;
```

第二，根据 `memoizedState` 判断是 mount 还是 update：

```ts
if (currentlyRenderingFiber!.memoizedState === null) {
  ReactSharedInternals.H = mountState;
} else {
  ReactSharedInternals.H = updateState;
}
```

如果函数组件 Fiber 上还没有 hook 链表：

```txt
memoizedState === null
```

说明是首次渲染，使用：

```ts
mountState
```

如果已经有 hook 链表，说明是更新渲染，使用：

```ts
updateState
```

第三，执行组件函数：

```ts
const result = Component();
```

组件内部如果调用 `useState`，就会走刚刚设置好的 dispatcher。

第四，清理全局变量：

```ts
workInProgressHook = null;
currentlyRenderingFiber = null;
```

最后返回组件的渲染结果，也就是 children。

---

## 6. mount 阶段：创建 hook

mount 阶段的入口是：

```ts
mountState(initialState)
```

源码：

```ts
export function mountState(initialState: any) {
  const hook = mountWorkInProgressHook(initialState);
  const dispatch = dispatchSetState.bind(null, currentlyRenderingFiber!, hook);
  hook.dispatch = dispatch;
  return [hook.memoizedState, dispatch];
}
```

它做三件事。

第一，创建 hook：

```ts
const hook = mountWorkInProgressHook(initialState);
```

第二，创建 dispatch：

```ts
const dispatch = dispatchSetState.bind(null, currentlyRenderingFiber!, hook);
```

这里用 `bind` 把当前 Fiber 和当前 hook 绑定进去。

所以以后调用：

```ts
setCount(1)
```

本质上会调用：

```ts
dispatchSetState(AppFiber, countHook, 1)
```

第三，返回状态和更新函数：

```ts
return [hook.memoizedState, dispatch];
```

---

## 7. `mountWorkInProgressHook`

源码：

```ts
function mountWorkInProgressHook(initialState: any) {
  const hook: Hook = {
    memoizedState: initialState,
    dispatch: null,
    next: null
  }

  if (workInProgressHook == null) {
    currentlyRenderingFiber!.memoizedState = hook;
  } else {
    workInProgressHook!.next = hook;
  }

  workInProgressHook = hook;
  return hook;
}
```

这个函数负责把多个 hook 串成链表。

如果这是第一个 hook：

```ts
currentlyRenderingFiber!.memoizedState = hook;
```

也就是：

```txt
App Fiber.memoizedState -> 第一个 hook
```

如果这不是第一个 hook：

```ts
workInProgressHook!.next = hook;
```

也就是：

```txt
上一个 hook.next -> 当前 hook
```

最后：

```ts
workInProgressHook = hook;
```

把当前 hook 记录为链表尾部。

---

## 8. mount 阶段例子

组件：

```tsx
function App() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState("du1");
  return <div>{count}</div>;
}
```

首次渲染时：

```txt
renderWihtHooks(App Fiber, App)
  ↓
currentlyRenderingFiber = App Fiber
  ↓
App Fiber.memoizedState === null
  ↓
ReactSharedInternals.H = mountState
  ↓
执行 App()
```

第一次 `useState(0)`：

```txt
mountState(0)
  ↓
创建 countHook
  ↓
App Fiber.memoizedState = countHook
  ↓
返回 [0, setCount]
```

第二次 `useState("du1")`：

```txt
mountState("du1")
  ↓
创建 nameHook
  ↓
countHook.next = nameHook
  ↓
返回 ["du1", setName]
```

最终：

```txt
App Fiber.memoizedState
  ↓
countHook
  memoizedState: 0
  dispatch: setCount
  next -> nameHook

nameHook
  memoizedState: "du1"
  dispatch: setName
  next -> null
```

---

## 9. update 阶段：读取已有 hook

更新阶段的入口是：

```ts
updateState()
```

源码：

```ts
export function updateState() {
  const hook = updateWorkInProgressHook();
  return [hook!.memoizedState, hook!.dispatch];
}
```

它不会重新创建 hook。

它只是找到当前顺序对应的 hook，然后返回：

```txt
hook.memoizedState
hook.dispatch
```

---

## 10. `updateWorkInProgressHook`

源码：

```ts
function updateWorkInProgressHook() {
  if (workInProgressHook === null) {
    workInProgressHook = currentlyRenderingFiber!.memoizedState;
  } else {
    workInProgressHook = workInProgressHook.next;
  }
  return workInProgressHook;
}
```

第一次调用 `useState` 时：

```ts
workInProgressHook = currentlyRenderingFiber!.memoizedState;
```

也就是取第一个 hook。

第二次调用 `useState` 时：

```ts
workInProgressHook = workInProgressHook.next;
```

也就是取第二个 hook。

所以 hooks 必须按固定顺序调用。

---

## 11. 为什么 hooks 不能写在条件里

看这个例子：

```tsx
function App({ visible }) {
  const [count, setCount] = useState(0);

  if (visible) {
    const [name, setName] = useState("du1");
  }

  const [age, setAge] = useState(18);
}
```

如果第一次渲染 `visible = true`，hook 顺序是：

```txt
countHook -> nameHook -> ageHook
```

如果第二次渲染 `visible = false`，hook 顺序变成：

```txt
countHook -> ageHook
```

但 update 阶段是按链表顺序取 hook 的：

```txt
第一次 useState -> 第一个 hook
第二次 useState -> 第二个 hook
```

这样 `age` 就可能读到原来的 `nameHook`。

所以 React 要求：

> Hooks 必须在函数组件顶层调用，不能写在条件、循环、嵌套函数里。

你的教学版虽然没有写规则校验，但底层机制已经能解释这个规则。

---

## 12. `setState` 做了什么

源码：

```ts
function dispatchSetState(fiber: Fiber, hook: Hook, newState: any) {
  hook.memoizedState = newState;
  updateOnFiber(fiber);
}
```

调用：

```ts
setCount(1)
```

会做两件事。

第一，更新 hook 上保存的状态：

```ts
hook.memoizedState = newState;
```

第二，触发当前 Fiber 更新：

```ts
updateOnFiber(fiber);
```

也就是说：

```txt
setState
  ↓
修改 hook.memoizedState
  ↓
updateOnFiber
  ↓
重新 render
  ↓
重新 commit
```

---

## 13. `updateOnFiber`

核心文件：

```txt
packages/reconciler/WorkLoop.ts
```

源码：

```ts
export function updateOnFiber(fiber: Fiber) {
  const hostRootFiber = getRootForUpdateFiber(fiber);
  removeChild(hostRootFiber.stateNode.containerInfo, hostRootFiber.child?.stateNode);
  workLoop(fiber);
  commitMutaionEffects(hostRootFiber);
}
```

更新流程是：

```txt
从当前 Fiber 向上找到 HostRootFiber
  ↓
删除旧 DOM
  ↓
重新执行 workLoop
  ↓
重新 commit
```

当前教学版的更新策略比较直接：

```txt
整棵子树重新 render
旧 DOM 删除
新 DOM 再挂载
```

真实 React 会更复杂，会使用 update queue、lane、effect flags、diff 复用等机制。

---

## 14. 为什么状态挂在 Fiber 上

函数组件本身只是函数。

比如：

```tsx
function App() {
  const [count, setCount] = useState(0);
  return <div>{count}</div>;
}
```

每次执行 `App()`，里面的局部变量都会重新创建。

所以状态不能保存在函数局部变量里。

React 需要一个函数之外、但又能代表这个组件实例的地方来保存状态。

这个地方就是：

```txt
App Fiber.memoizedState
```

也就是说：

> 函数组件的状态不是保存在函数里，而是保存在对应的 Fiber 上。

---

## 15. 当前实现和真实 React 的差异

当前教学版已经实现了 hooks 的核心骨架：

- 通过 dispatcher 区分 mount/update
- 通过 Fiber 保存 hook 链表
- 通过 hook 保存 state 和 dispatch
- 通过调用顺序匹配不同 hook
- 通过 `setState` 触发重新渲染

但它还没有实现真实 React 中的一些复杂机制：

- update queue
- 函数式更新，比如 `setCount(c => c + 1)`
- 批处理更新
- lane 优先级
- eager state
- render phase update
- 多次 setState 合并
- bailout 跳过无变化更新

当前实现里：

```ts
hook.memoizedState = newState;
```

是直接把新状态写进去。

这很适合教学，因为能直接看到：

```txt
hook 保存状态
setState 修改状态
Fiber 重新渲染
```

---

## 16. 一个完整流程

组件：

```tsx
function App() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(1)}>{count}</button>;
}
```

首次渲染：

```txt
beginWork(App Fiber)
  ↓
renderWihtHooks(App Fiber, App)
  ↓
ReactSharedInternals.H = mountState
  ↓
执行 App()
  ↓
useState(0)
  ↓
创建 hook
  ↓
App Fiber.memoizedState = hook
  ↓
返回 [0, setCount]
  ↓
返回 button ReactElement
```

点击按钮：

```txt
setCount(1)
  ↓
dispatchSetState(App Fiber, hook, 1)
  ↓
hook.memoizedState = 1
  ↓
updateOnFiber(App Fiber)
```

更新渲染：

```txt
renderWihtHooks(App Fiber, App)
  ↓
App Fiber.memoizedState !== null
  ↓
ReactSharedInternals.H = updateState
  ↓
执行 App()
  ↓
useState(0)
  ↓
读取已有 hook
  ↓
返回 [1, setCount]
  ↓
返回新的 button ReactElement
  ↓
workLoop + commit
```

最后页面显示从：

```txt
0
```

变成：

```txt
1
```

---

## 17. 本章总结

这一章最重要的一句话：

> `useState` 的状态保存在函数组件 Fiber 的 hook 链表上，`setState` 修改 hook 状态后触发当前 Fiber 重新渲染。

可以拆成几句：

- `ReactSharedInternals.H` 是当前 hooks dispatcher。
- mount 阶段 `useState` 会走 `mountState`。
- update 阶段 `useState` 会走 `updateState`。
- 每个 hook 保存 `memoizedState`、`dispatch`、`next`。
- 多个 hooks 通过 `next` 串成链表。
- hook 链表挂在函数组件 Fiber 的 `memoizedState` 上。
- hooks 依赖调用顺序，所以不能写在条件或循环里。
- `setState` 会修改 hook 状态，并调用 `updateOnFiber` 触发更新。

整体流程是：

```txt
FunctionComponent Fiber
  ↓
renderWihtHooks
  ↓
设置 dispatcher
  ↓
执行组件函数
  ↓
useState
  ↓
mountState 或 updateState
  ↓
hook 链表挂到 Fiber.memoizedState
  ↓
setState
  ↓
updateOnFiber
  ↓
重新 render + commit
```

下一章可以继续看 **Child Reconciliation：子节点如何变成 Fiber 链表**。


