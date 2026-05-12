---
title: React-05
date: 2026-05-12 11:37:345
categories:
  - ReactCore
tags:
  - render 阶段
---

# 第 4 讲：render 阶段：workLoop、beginWork、completeWork

核心文件：

- `packages/reconciler/WorkLoop.ts`
- `packages/reconciler/BeginWork.ts`
- `packages/reconciler/CompleteWork.ts`
- `packages/reconciler/ChildFiber.ts`

这一讲要回答的问题是：

> Fiber 创建出来以后，React 是怎么遍历 Fiber 树，并创建真实 DOM 的？

前面几章的主线是：

```txt
JSX
  ↓
ReactElement
  ↓
root.render
  ↓
createFiberFromElement
  ↓
Fiber
```

从这一章开始，Fiber 会进入真正的 render 阶段。

在当前教学版实现里，render 阶段主要做两件事：

- 向下遍历 Fiber，创建子 Fiber。
- 向上回溯 Fiber，创建真实 DOM，并把子 DOM 挂到父 DOM 上。

---

## 1. render 阶段是什么

render 阶段可以理解成：

> 根据 ReactElement/Fiber，计算出一棵完整的 Fiber 树，并在回溯阶段创建好对应的 DOM 节点。

注意，这里还没有把 DOM 插入页面容器。

真正插入页面是在 commit 阶段：

```ts
commitMutaionEffects(root);
```

所以整体可以先分成：

```txt
render 阶段：构建 Fiber，创建 DOM
commit 阶段：把 DOM 插入页面
```

这一章只看 render 阶段。

---

## 2. render 阶段入口：`workLoop`

源码：

```ts
let workInProgress: Fiber | null = null;

export function workLoop(fiber: Fiber) {
  workInProgress = fiber;
  while (workInProgress) {
    performUnitOfWork(workInProgress)
  }
}
```

`workInProgress` 表示：

> 当前正在处理的 Fiber。

`workLoop` 做的事情非常直接：

```txt
只要还有正在处理的 Fiber
  就执行 performUnitOfWork
```

也就是说：

```txt
workLoop
  ↓
performUnitOfWork
  ↓
beginWork 或 completeWork
```

---

## 3. 一个 Fiber 是一个工作单元

源码：

```ts
function performUnitOfWork(fiber: Fiber) {
  let next = beginWork(fiber)
  if (next) {
    workInProgress = next;
  } else {
    completeUnitOfWork(fiber);
  }
}
```

每处理一个 Fiber，大致分两步：

第一步，先执行：

```ts
beginWork(fiber)
```

如果 `beginWork` 返回了子 Fiber：

```ts
workInProgress = next;
```

说明要继续向下走。

如果没有子 Fiber：

```ts
completeUnitOfWork(fiber);
```

说明当前 Fiber 已经走到底了，要开始回溯。

可以理解成：

```txt
有 child：继续向下
没 child：开始完成当前节点，并向上找 sibling 或 parent
```

---

## 4. `beginWork`：向下阶段

源码：

```ts
export function beginWork(fiber: Fiber): Fiber | null {
  if (
    typeof fiber.pendingProps.children === 'string' ||
    typeof fiber.pendingProps.children === 'number'
  ) {
    return null;
  }

  switch (fiber.tag) {
    case HostRoot:
      return null;
    case FunctionComponent:
      const children = renderWihtHooks(fiber, fiber.type);
      fiber.child = reconcileChildFibers(fiber, children)
      return fiber.child;
    case HostComponent:
      fiber.child = reconcileChildFibers(fiber, fiber.pendingProps.children);
      return fiber.child;
    default:
      return null;
  }
}
```

`beginWork` 的职责是：

> 处理当前 Fiber，并返回它的第一个子 Fiber。

如果返回了子 Fiber，`workLoop` 就会继续往下处理子 Fiber。

如果返回 `null`，说明当前节点没有可继续向下处理的子节点。

---

## 5. 为什么文本 children 会直接返回 `null`

源码一开始有这段：

```ts
if (
  typeof fiber.pendingProps.children === 'string' ||
  typeof fiber.pendingProps.children === 'number'
) {
  return null;
}
```

比如：

```tsx
<div>hello</div>
```

`div Fiber` 的 `pendingProps.children` 是字符串 `"hello"`。

当前实现里，这种文本 children 没有在 `beginWork` 中创建 `HostText Fiber`，而是在 `completeWork` 设置 DOM 属性时处理：

```ts
dom.textContent = props.children;
```

所以对于纯文本子节点，`beginWork` 会直接返回 `null`。

注意：项目里也有 `createFiberFromText`，数组 children 中的文本会走文本 Fiber；但单个字符串 children 在当前代码里是通过 `textContent` 设置的。

---

## 6. 函数组件的 `beginWork`

源码：

```ts
case FunctionComponent:
  const children = renderWihtHooks(fiber, fiber.type);
  fiber.child = reconcileChildFibers(fiber, children)
  return fiber.child;
```

函数组件本身不会直接创建 DOM。

比如：

```tsx
function App() {
  return <div>hello</div>
}
```

处理 `App Fiber` 时，需要先执行组件函数：

```ts
const children = App();
```

在你的项目里，这一步封装在：

```ts
renderWihtHooks(fiber, fiber.type)
```

它不仅会执行函数组件，还会准备 hooks 环境。

函数组件返回的 JSX 会变成 ReactElement，然后通过：

```ts
reconcileChildFibers(fiber, children)
```

创建子 Fiber。

所以函数组件的流程是：

```txt
FunctionComponent Fiber
  ↓
执行组件函数
  ↓
得到 children ReactElement
  ↓
reconcileChildFibers
  ↓
创建 child Fiber
```

---

## 7. 原生 DOM 节点的 `beginWork`

源码：

```ts
case HostComponent:
  fiber.child = reconcileChildFibers(fiber, fiber.pendingProps.children);
  return fiber.child;
```

原生 DOM 节点，比如：

```tsx
<div>
  <span>hello</span>
</div>
```

`div Fiber` 的 children 是：

```tsx
<span>hello</span>
```

所以 `beginWork(div Fiber)` 会调用：

```ts
reconcileChildFibers(divFiber, divFiber.pendingProps.children)
```

创建 `span Fiber`，并挂到：

```ts
divFiber.child
```

上。

---

## 8. `reconcileChildFibers`：创建子 Fiber

核心文件：

```txt
packages/reconciler/ChildFiber.ts
```

源码：

```ts
export function reconcileChildFibers(fiber: Fiber, children: any): Fiber | null {
  if (children.$$typeof === REACT_ELEMENT_TYPE) {
    return reconcileSingleElement(fiber, children);
  }

  if (Array.isArray(children)) {
    return reconcileChildrenArray(fiber, children);
  }

  return null;
}
```

当前教学版主要处理两种情况：

- 单个 ReactElement
- ReactElement 数组

单节点时：

```ts
function reconcileSingleElement(fiber: any, children: any): Fiber {
  const created = createFiberFromElement(children);
  created.return = fiber;
  return created;
}
```

它会创建一个子 Fiber，并把子 Fiber 的父指针指回来：

```txt
child.return -> parent
```

数组节点时：

```ts
previousNewFiber.sibling = newFiber;
```

它会用 `sibling` 把兄弟 Fiber 串起来：

```txt
firstChild
  sibling -> secondChild
              sibling -> thirdChild
```

这就是第三章讲过的 Fiber 树结构真正建立起来的地方。

---

## 9. `completeWork`：回溯阶段

当某个 Fiber 没有子节点可以继续向下时，会进入：

```ts
completeUnitOfWork(fiber)
```

然后调用：

```ts
completeWork(completedWork);
```

`completeWork` 的职责是：

> 当前 Fiber 的子节点都处理完后，为当前 Fiber 创建真实 DOM，并把子 DOM 挂到当前 DOM 上。

源码：

```ts
export function completeWork(fiber: Fiber) {
  switch (fiber.tag) {
    case HostText:
      fiber.stateNode = createTextInstance(fiber.pendingProps);
      break;
    case FunctionComponent:
      break;
    case HostComponent:
      const instance = createDom(fiber.type, fiber);
      appendAllChildren(instance, fiber.child);
      setInitialProps(instance, fiber.pendingProps);
      fiber.stateNode = instance
      break
  }
}
```

不同 Fiber 类型做的事情不一样。

---

## 10. `HostText` 的完成工作

源码：

```ts
case HostText:
  fiber.stateNode = createTextInstance(fiber.pendingProps);
  break;
```

文本 Fiber 会创建真实文本节点：

```ts
document.createTextNode(text)
```

然后保存到：

```ts
fiber.stateNode
```

---

## 11. `FunctionComponent` 的完成工作

源码：

```ts
case FunctionComponent:
  break;
```

函数组件本身不创建 DOM。

它只是执行函数，得到子元素。

所以它的真实 DOM 来自它的子 Fiber。

比如：

```tsx
function App() {
  return <div>hello</div>
}
```

`App Fiber` 没有自己的 DOM。

真正的 DOM 是 `div Fiber.stateNode`。

---

## 12. `HostComponent` 的完成工作

源码：

```ts
case HostComponent:
  const instance = createDom(fiber.type, fiber);
  appendAllChildren(instance, fiber.child);
  setInitialProps(instance, fiber.pendingProps);
  fiber.stateNode = instance
  break
```

原生 DOM 节点的完成工作分四步：

第一，创建真实 DOM：

```ts
const instance = createDom(fiber.type, fiber);
```

比如 `fiber.type` 是 `"div"`，就会创建：

```ts
document.createElement("div")
```

第二，把子 DOM 挂到当前 DOM 上：

```ts
appendAllChildren(instance, fiber.child);
```

第三，设置初始属性：

```ts
setInitialProps(instance, fiber.pendingProps);
```

第四，保存真实 DOM：

```ts
fiber.stateNode = instance;
```

---

## 13. `appendAllChildren`

源码：

```ts
function appendAllChildren(parent: Instance, child: Fiber | null) {
  let node: Fiber | null = child;
  while (node) {
    let childStateNode =
      node.tag === FunctionComponent ? node.child?.stateNode : node.stateNode;
    appendChild(parent, childStateNode);
    node = node.sibling;
  }
}
```

它的作用是：

> 把当前 Fiber 的所有子 DOM 添加到当前 DOM 里。

例如：

```tsx
<div>
  <span>A</span>
  <p>B</p>
</div>
```

当 `div Fiber` 进入 `completeWork` 时：

```txt
div.child -> span
span.sibling -> p
```

`appendAllChildren` 会遍历：

```txt
span -> p
```

把它们的 `stateNode` 添加到 `div DOM` 里。

特别注意这句：

```ts
node.tag === FunctionComponent ? node.child?.stateNode : node.stateNode
```

因为函数组件本身没有 DOM，所以如果子节点是函数组件，就取它第一个子 Fiber 的 DOM。

---

## 14. `completeUnitOfWork`：怎么回溯

源码：

```ts
function completeUnitOfWork(fiber: Fiber) {
  let completedWork: Fiber | null = fiber
  do {
    completeWork(completedWork);
    if (completedWork.sibling) {
      workInProgress = completedWork.sibling;
      return;
    }

    completedWork = completedWork.return;
    workInProgress = completedWork;
  } while (completedWork)
}
```

这段是 render 阶段最精妙的地方。

当前节点完成后，有两种情况。

第一，如果有兄弟节点：

```ts
if (completedWork.sibling) {
  workInProgress = completedWork.sibling;
  return;
}
```

就转向兄弟节点继续处理。

第二，如果没有兄弟节点：

```ts
completedWork = completedWork.return;
```

就回到父节点，继续完成父节点。

所以遍历顺序是：

```txt
先一路向下找 child
到底后 complete 当前节点
有 sibling 就处理 sibling
没有 sibling 就 return 回父节点
```

这就是 Fiber 的深度优先遍历。

---

## 15. 一个完整遍历例子

假设 JSX 是：

```tsx
<div>
  <span>A</span>
  <p>B</p>
</div>
```

Fiber 结构是：

```txt
div
  child -> span
             sibling -> p
```

render 阶段大致顺序是：

```txt
beginWork(div)
  创建 span / p Fiber
  返回 span

beginWork(span)
  文本 children，返回 null

completeWork(span)
  创建 span DOM
  设置 textContent = "A"
  发现 sibling 是 p，转向 p

beginWork(p)
  文本 children，返回 null

completeWork(p)
  创建 p DOM
  设置 textContent = "B"
  没有 sibling，回到 div

completeWork(div)
  创建 div DOM
  append span DOM
  append p DOM
  保存 div.stateNode
```

最后 render 阶段结束时：

```txt
div Fiber.stateNode -> div DOM
span Fiber.stateNode -> span DOM
p Fiber.stateNode -> p DOM
```

但是 `div DOM` 还没有挂到页面容器里。

挂到页面容器是下一步 commit 阶段做的。

---

## 16. render 阶段和 commit 阶段的边界

当前项目里：

```ts
workLoop(containerFiber);
```

执行 render 阶段。

然后：

```ts
commitMutaionEffects(root);
```

执行 commit 阶段。

render 阶段结束后，DOM 已经创建好了：

```txt
Fiber.stateNode -> DOM
```

但页面还没有更新。

commit 阶段会做：

```ts
appendChild(fiber.stateNode.containerInfo, fiber.child?.stateNode);
```

也就是把应用根 DOM 插入真实页面容器。

---

## 17. 本章总结

这一章最重要的一句话：

> render 阶段用深度优先遍历处理 Fiber：`beginWork` 向下创建子 Fiber，`completeWork` 回溯创建 DOM。

可以拆成几句：

- `workLoop` 保存当前正在处理的 Fiber。
- `performUnitOfWork` 先执行 `beginWork`。
- `beginWork` 负责根据 children 创建子 Fiber。
- 如果有子 Fiber，就继续向下。
- 如果没有子 Fiber，就进入 `completeUnitOfWork`。
- `completeWork` 负责创建真实 DOM，并把子 DOM 挂到父 DOM。
- 有兄弟 Fiber 就转向兄弟，没有兄弟就回到父 Fiber。

整体流程是：

```txt
workLoop
  ↓
performUnitOfWork
  ↓
beginWork 当前 Fiber
  ↓
有 child：继续向下
  ↓
没有 child：completeWork 当前 Fiber
  ↓
有 sibling：处理 sibling
  ↓
没有 sibling：return 回父 Fiber
```

下一章可以继续看 **commit 阶段：把 render 阶段创建好的 DOM 挂载到页面上**。


