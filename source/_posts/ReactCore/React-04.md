---
title: React-03
date: 2026-05-12 11:13:32
categories:
  - ReactCore
tags:
  - Fiber 数据结构
---

# 第 3 讲：Fiber 数据结构

核心文件：

- `packages/reconciler/Fiber.ts`
- `packages/reconciler/ReactInternalTyes.ts`
- `packages/reconciler/FiberRoot.ts`

这一讲要回答的问题是：

> ReactElement 进入 reconciler 之后，为什么要先变成 Fiber？

上一章我们看到：

```ts
root.render(element)
```

最终会调用：

```ts
updateContainer(element, hostRootFiber)
```

然后第一步就是：

```ts
const containerFiber = createFiberFromElement(element);
```

也就是说：

```txt
ReactElement
  ↓
Fiber
```

ReactElement 是 UI 的静态描述，而 Fiber 是 React 内部真正用来组织、遍历、更新和挂载的工作单元。

---

## 1. Fiber 是什么

可以先用一句话理解：

> Fiber 是 React 内部的工作节点，每一个 Fiber 大致对应一个组件、一个 DOM 节点、一个文本节点，或者一个根节点。

比如这段 JSX：

```tsx
<div>
  <span>hello</span>
  <p>world</p>
</div>
```

会大致形成这样的 Fiber 结构：

```txt
div Fiber
  child -> span Fiber
             sibling -> p Fiber
```

`span` 下面还会有文本节点：

```txt
span Fiber
  child -> "hello" HostText Fiber
```

Fiber 不只是描述 UI，它还会保存：

- 当前节点是什么类型
- 它对应的 DOM 是什么
- 它的父节点是谁
- 它的第一个子节点是谁
- 它的兄弟节点是谁
- 它本次渲染要用的 props
- 它的 hook 状态

---

## 2. Fiber 类型定义

源码：

```ts
export type Fiber = {
  tag: WorkTag,
  key: string | null,
  elementType: any,
  type: any,
  stateNode: any,
  return: Fiber | null,
  child: Fiber | null,
  sibling: Fiber | null,
  ref: any,
  pendingProps: any,
  memoizedState: any,
}
```

这是整个 mini React 里最核心的数据结构之一。

后面的 render 阶段、commit 阶段、hooks、事件系统，都会围绕 Fiber 展开。

---

## 3. `tag`：Fiber 节点类型

源码：

```ts
export type WorkTag = 0 | 3 | 5 | 6;
export const FunctionComponent = 0;
export const HostRoot = 3;
export const HostComponent = 5;
export const HostText = 6;
```

`tag` 用来表示这个 Fiber 是哪种节点。

当前项目里有四种：

```txt
FunctionComponent -> 函数组件
HostRoot          -> 根 Fiber
HostComponent     -> 原生 DOM 节点，比如 div、span
HostText          -> 文本节点
```

比如：

```tsx
function App() {
  return <div>hello</div>
}
```

大致对应：

```txt
App Fiber       -> FunctionComponent
div Fiber       -> HostComponent
"hello" Fiber   -> HostText
```

---

## 4. `type` 和 `elementType`

在你的实现里：

```ts
fiber.elementType = type;
fiber.type = type;
```

对于原生 DOM：

```tsx
<div />
```

会得到：

```ts
type: "div"
elementType: "div"
```

对于函数组件：

```tsx
<App />
```

会得到：

```ts
type: App
elementType: App
```

当前教学版里 `type` 和 `elementType` 基本相同。

在真实 React 中，它们在 `memo`、`forwardRef`、热更新等场景下会有区别；但在这个项目里，可以先理解为：

> `type` 表示这个 Fiber 对应的组件函数或 DOM 标签名。

---

## 5. `stateNode`

`stateNode` 表示 Fiber 对应的真实实例。

不同类型的 Fiber，`stateNode` 含义不一样。

对于 `HostComponent`：

```tsx
<div />
```

`stateNode` 是真实 DOM：

```ts
HTMLDivElement
```

对于 `HostText`：

```tsx
hello
```

`stateNode` 是真实文本节点：

```ts
Text
```

对于 `HostRoot`：

```ts
hostRootFiber.stateNode = root;
```

`stateNode` 指向 `FiberRoot`：

```txt
HostRootFiber.stateNode -> FiberRoot
```

而 `FiberRoot` 里保存真实容器：

```ts
export type FiberRoot = {
  containerInfo: HTMLElement
}
```

所以根部关系是：

```txt
HostRootFiber.stateNode -> FiberRoot.containerInfo -> 真实 DOM 容器
```

---

## 6. `return`、`child`、`sibling`

这三个字段用来把 Fiber 组织成树。

源码：

```ts
return: Fiber | null,
child: Fiber | null,
sibling: Fiber | null,
```

含义是：

```txt
return  -> 父 Fiber
child   -> 第一个子 Fiber
sibling -> 下一个兄弟 Fiber
```

React Fiber 不是用普通的 `children: []` 数组保存树结构，而是使用：

```txt
父 -> 第一个子
子 -> 下一个兄弟
子 -> 父
```

也就是：

```txt
parent.child
child.sibling
child.return
```

比如 JSX：

```tsx
<div>
  <span>A</span>
  <p>B</p>
</div>
```

对应 Fiber 链接大致是：

```txt
div Fiber
  child -> span Fiber
             sibling -> p Fiber

span.return -> div
p.return    -> div
```

这种结构非常适合后面的深度优先遍历。

---

## 7. `pendingProps`

`pendingProps` 表示本次渲染传入的新 props。

创建 Fiber 时：

```ts
fiber.pendingProps = pendingProps;
```

比如：

```tsx
<button id="save">Save</button>
```

这个 `button Fiber` 的 `pendingProps` 大概是：

```ts
{
  id: "save",
  children: "Save"
}
```

后面 `beginWork` 会读取：

```ts
fiber.pendingProps.children
```

继续创建子 Fiber。

`completeWork` 会读取：

```ts
setInitialProps(instance, fiber.pendingProps);
```

把属性设置到真实 DOM 上。

---

## 8. `memoizedState`

`memoizedState` 用来保存状态。

在当前项目里，它主要服务于 hooks。

函数组件 Fiber 上：

```ts
fiber.memoizedState
```

会指向 hook 链表的第一个 hook。

比如：

```tsx
function App() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState("du1");
}
```

大致会形成：

```txt
App Fiber.memoizedState
  ↓
hook(count)
  ↓ next
hook(name)
```

所以 `memoizedState` 可以理解成：

> Fiber 身上保存的内部状态。

这一点在 hooks 章节会非常关键。

---

## 9. `key` 和 `ref`

`key` 来自 ReactElement：

```tsx
<li key="a">A</li>
```

创建 Fiber 时会保存：

```ts
fiber.key = key;
```

它主要用于后续 children diff。

当前教学版里还没有完整 diff 复用逻辑，所以 `key` 暂时只是被保存下来。

`ref` 在 Fiber 上也预留了字段：

```ts
ref: any
```

不过当前项目还没有完整实现 ref 的挂载和更新。

---

## 10. 创建普通 Fiber

源码：

```ts
export function createFiber(tag: WorkTag, key: string | null): Fiber {
  const fiber: Fiber = {
    tag,
    key,
    elementType: null,
    type: null,
    stateNode: null,
    return: null,
    child: null,
    sibling: null,
    ref: null,
    pendingProps: null,
    memoizedState: null
  }
  return fiber;
}
```

`createFiber` 创建的是一个空 Fiber 骨架。

一开始很多字段都是 `null`，后续不同阶段会慢慢填充：

- 创建阶段填 `tag`、`key`、`type`、`pendingProps`
- begin 阶段填 `child`
- complete 阶段填 `stateNode`
- hooks 阶段填 `memoizedState`
- reconciliation 阶段填 `return`、`sibling`

---

## 11. 从 ReactElement 创建 Fiber

源码：

```ts
export function createFiberFromElement(element: ReactElement): Fiber {
  const { type, props, key } = element;
  const fiber: Fiber = createFiberFromTypeAndProps(type, props, key);
  return fiber;
}
```

它把 ReactElement 中的：

```ts
type
props
key
```

转移到 Fiber 上。

核心逻辑在这里：

```ts
export function createFiberFromTypeAndProps(
  type: any,
  pendingProps: any,
  key: string | null
): Fiber {
  let fiberTag: WorkTag =
    typeof type === 'function' ? FunctionComponent : HostComponent;

  const fiber = createFiber(fiberTag, key);
  fiber.elementType = type;
  fiber.type = type;
  fiber.pendingProps = pendingProps;
  return fiber;
}
```

判断规则很简单：

```txt
typeof type === "function" -> FunctionComponent
否则                         -> HostComponent
```

所以：

```tsx
<App />
```

会创建 `FunctionComponent Fiber`。

```tsx
<div />
```

会创建 `HostComponent Fiber`。

---

## 12. 创建文本 Fiber

源码：

```ts
export function createFiberFromText(text: string): Fiber {
  const fiber = createFiber(HostText, null);
  fiber.pendingProps = text;
  return fiber;
}
```

文本节点没有 `type`，也没有 `key`。

它的内容直接放在：

```ts
fiber.pendingProps
```

比如：

```tsx
<div>hello</div>
```

`hello` 会变成：

```ts
{
  tag: HostText,
  pendingProps: "hello"
}
```

---

## 13. 创建 HostRootFiber

源码：

```ts
export function createHostRootFiber(): Fiber {
  const fiber = createFiber(HostRoot, null);
  return fiber;
}
```

`HostRootFiber` 是整棵 Fiber 树的根节点。

它不直接对应页面上的某个组件，而是 React 内部用来承载整个应用树的根。

在 `createContainer` 里：

```ts
const root = createFiberRoot(containerInfo);
const hostRootFiber = createHostRootFiber();
hostRootFiber.stateNode = root;
```

形成：

```txt
HostRootFiber
  stateNode -> FiberRoot
                 containerInfo -> 真实 DOM 容器
```

后续 `root.render(<App />)` 时：

```ts
root.child = containerFiber;
containerFiber.return = root;
```

形成：

```txt
HostRootFiber
  child -> App Fiber
```

---

## 14. 一个完整例子

假设 JSX 是：

```tsx
function App() {
  return (
    <div id="app">
      <span>hello</span>
      <p>world</p>
    </div>
  );
}

root.render(<App />);
```

大致 Fiber 结构是：

```txt
HostRootFiber
  child -> App Fiber(FunctionComponent)
             child -> div Fiber(HostComponent)
                        child -> span Fiber(HostComponent)
                                   child -> "hello" Fiber(HostText)
                        sibling? no

span Fiber
  sibling -> p Fiber(HostComponent)
                child -> "world" Fiber(HostText)
```

如果只看核心指针：

```txt
HostRootFiber.child -> App
App.return          -> HostRootFiber

App.child           -> div
div.return          -> App

div.child           -> span
span.return         -> div
span.sibling        -> p
p.return            -> div
```

这就是 Fiber 树最重要的结构。

---

## 15. 本章总结

这一章最重要的一句话：

> Fiber 是 React 内部的工作节点，它把 ReactElement 变成可遍历、可更新、可保存状态的树形链表结构。

可以再拆成几句：

- ReactElement 是静态 UI 描述。
- Fiber 是 React 内部的工作单元。
- `tag` 表示 Fiber 类型。
- `stateNode` 指向真实实例，比如 DOM、Text 或 FiberRoot。
- `return`、`child`、`sibling` 组成 Fiber 树。
- `pendingProps` 保存本次渲染的 props。
- `memoizedState` 保存 hooks 等内部状态。

整体链路是：

```txt
ReactElement
  ↓
createFiberFromElement
  ↓
Fiber
  ↓
workLoop 遍历
  ↓
completeWork 创建 DOM
  ↓
commit 挂载 DOM
```

下一章可以继续看 **render 阶段：workLoop、beginWork、completeWork**。



