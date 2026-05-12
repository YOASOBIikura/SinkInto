---
title: React-02
date: 2026-05-12 10:39:00
categories:
  - ReactCore
tags:
  - React Element
---


# 第 1 讲：JSX 到 ReactElement

核心文件：

- `packages/react/jsx-runtime.ts`

这一讲要回答一个问题：

> JSX 到底是什么？为什么 `<div>hello</div>` 最后能被 React 渲染？

答案是：

> JSX 本身不是运行时能直接理解的东西，它会被编译成函数调用，函数调用再返回一个普通 JS 对象，也就是 ReactElement。

---

## 1. JSX 会被编译成函数调用

比如你写：

```tsx
<div id="app">hello</div>
```

经过 JSX 编译后，大概会变成：

```ts
jsx("div", {
  id: "app",
  children: "hello"
})
```

而项目里的 `jsx` 函数会返回这样的对象：

```ts
{
  $$typeof: Symbol.for("du1React"),
  type: "div",
  props: {
    id: "app",
    children: "hello"
  },
  key: null,
  ref: null
}
```

这就是 **ReactElement**。

它不是 DOM。

它不是 Fiber。

它只是一个 **描述 UI 的普通 JS 对象**。

---

## 2. ReactElement 的结构

源码：

```ts
function ReactElement(type: any, props: any, key: any, ref: any): ReactElement {
  return {
    $$typeof: typeof Symbol === 'function' && Symbol.for
      ? Symbol.for('du1React')
      : 'du1React',
    type,
    props,
    key,
    ref
  }
}
```

ReactElement 主要有这几个字段：

```ts
{
  $$typeof,
  type,
  props,
  key,
  ref
}
```

---

## 3. `$$typeof`

`$$typeof` 用来标记：

> 这是一个合法的 ReactElement。

项目里用的是：

```ts
Symbol.for("du1React")
```

React 官方源码里也有类似设计，它会用一个特殊 symbol 来识别 ReactElement，避免普通对象被误认为 React 元素。

对应文件：

```txt
packages/shared/ReactSymbols.ts
```

---

## 4. `type`

`type` 表示元素类型。

不同 JSX 会得到不同的 `type`。

比如：

```tsx
<div />
```

会得到：

```ts
type: "div"
```

而：

```tsx
<App />
```

会得到：

```ts
type: App
```

所以后面的 Fiber 创建阶段可以根据 `type` 判断这是函数组件还是原生 DOM 标签。

对应源码：

```ts
let fiberTag: WorkTag =
  typeof type === 'function' ? FunctionComponent : HostComponent;
```

对应文件：

```txt
packages/reconciler/Fiber.ts
```

也就是说：

- 如果 `type` 是函数，比如 `App`，就是函数组件。
- 如果 `type` 是字符串，比如 `"div"`、`"span"`，就是原生 DOM 标签。

---

## 5. `props`

`props` 表示 JSX 上的属性和 children。

比如：

```tsx
<button id="save" onClick={handleClick}>
  Save
</button>
```

会变成类似：

```ts
{
  type: "button",
  props: {
    id: "save",
    onClick: handleClick,
    children: "Save"
  },
  key: null,
  ref: null
}
```

注意：

> `children` 也是 `props` 的一部分。

这点很重要，因为后面调和子节点时，就是从：

```ts
fiber.pendingProps.children
```

里继续往下创建子 Fiber。

---

## 6. `key`

项目里的 `jsx` 函数会单独处理 `key`：

```ts
if (maybeKey !== undefined) {
  key = '' + maybeKey;
}

if (config.key !== undefined) {
  key = '' + config.key;
}
```

也就是说：

```tsx
<li key="a">A</li>
```

会把 `key` 单独提出来，不放进 `props`。

这和 React 的真实行为一致：

> `key` 是 React 内部用来做子节点 diff 的，不是普通 props。

---

## 7. `ref`

项目里的实现也会把 `ref` 单独提出来：

```ts
const ref = config.ref ? config.ref : null;
```

同样，`ref` 也不是普通 props，它是 React 内部特殊字段。

不过当前教学版里只是保存了 `ref`，还没有真正实现 ref 挂载逻辑。

---

## 8. 一个完整例子

```tsx
const element = <div id="app">hello</div>
```

本质上等价于创建了一个对象：

```ts
const element = {
  $$typeof: Symbol.for("du1React"),
  type: "div",
  props: {
    id: "app",
    children: "hello"
  },
  key: null,
  ref: null
}
```

React 后续所有工作，都是围绕这个对象展开。

---

## 9. 整体流程

```txt
JSX
  ↓
jsx(...)
  ↓
ReactElement 普通对象
  ↓
createFiberFromElement(...)
  ↓
Fiber
  ↓
DOM
```

---

## 10. 本讲总结

ReactElement 可以理解成：

> UI 的静态描述书。

它只描述“我要什么”，但还不负责“怎么创建、怎么更新、怎么插入页面”。

这些是后面 Fiber、Reconciler、Commit 阶段负责的。

本讲最重要的一句话：

> JSX 最终会变成 ReactElement，而 ReactElement 只是一个描述 UI 的普通 JS 对象。





