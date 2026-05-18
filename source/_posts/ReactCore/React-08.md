---
title: React-08
date: 2026-05-18 17:57:33
categories:
  - ReactCore
tags:
  - Fiber双缓存
---

# 第 9 讲：Fiber 双缓存、alternate 和 current 切换

核心文件：

- `packages/reconciler/Fiber.ts`
- `packages/reconciler/FiberRoot.ts`
- `packages/reconciler/FiberReconciler.ts`
- `packages/reconciler/WorkLoop.ts`
- `packages/reconciler/ChildFiber.ts`
- `packages/reconciler/FiberHook.ts`

这一讲要回答的问题是：

> React 为什么需要两棵 Fiber 树？更新时 current、workInProgress、alternate 是怎么配合的？

前面几章里，我们已经能完成：

```txt
ReactElement
  ↓
Fiber
  ↓
render 阶段
  ↓
commit 阶段
  ↓
真实 DOM
```

但如果每次更新都从零创建一整棵 Fiber，就没法复用之前的状态，也很难对比新旧节点。

所以这一章开始进入 Fiber 更新机制里很重要的一块：

```txt
双缓存 Fiber 树
```

---

## 1. 什么是双缓存

双缓存可以先用一句话理解：

> 页面上已经生效的是 current 树，正在内存里计算下一次更新的是 workInProgress 树。

也就是说，React 更新时不会直接改当前页面对应的 Fiber 树，而是先创建或复用另一棵树：

```txt
current Fiber tree         当前页面正在使用的树
workInProgress Fiber tree  正在计算中的新树
```

等 workInProgress 树计算完成并 commit 之后，再把它切换成新的 current 树。

这个过程很像：

```txt
旧画面还在屏幕上
新画面先在后台画好
画好以后一次性切过去
```

---

## 2. 项目里的关键字段

### `FiberRoot.current`

源码：

```ts
export type FiberRoot = {
  containerInfo: HTMLElement,
  current: Fiber | null,
}
```

`FiberRoot.current` 指向当前已经提交的 `HostRootFiber`。

可以理解成：

```txt
FiberRoot.current -> 当前页面对应的根 Fiber
```

首次创建容器时：

```ts
export function createContainer(containerInfo: HTMLElement) {
  const fiberRoot = createFiberRoot(containerInfo);
  const hostRootFiber = createHostRootFiber();
  hostRootFiber.memoizedState = { element: null };
  hostRootFiber.stateNode = fiberRoot;
  fiberRoot.current = hostRootFiber;
  return fiberRoot;
}
```

形成的关系是：

```txt
FiberRoot
  current -> HostRootFiber

HostRootFiber
  stateNode -> FiberRoot
  memoizedState -> { element: null }
```

这里要注意：

```txt
FiberRoot 不是 Fiber
HostRootFiber 才是 Fiber 树的根节点
```

---

### `Fiber.alternate`

源码：

```ts
export type Fiber = {
  alternate: Fiber | null,
  // ...
}
```

`alternate` 用来连接同一个节点的两份 Fiber：

```txt
current fiber <-> workInProgress fiber
```

比如同一个 `App` 组件，在 current 树上有一个 `App Fiber`，在 workInProgress 树上也有一个 `App Fiber`。

它们之间通过 `alternate` 互相指向：

```txt
current App Fiber.alternate -> workInProgress App Fiber
workInProgress App Fiber.alternate -> current App Fiber
```

这样更新时就能做到：

- 从旧 Fiber 读取已有状态。
- 在新 Fiber 上计算新结果。
- 提交后把新 Fiber 变成 current。

---

## 3. `createWorkInProgress`

双缓存的核心创建逻辑在这里：

```ts
export function createWorkInProgress(current: Fiber, pendingProps: any): Fiber {
  let workInProgress = current.alternate;

  if (workInProgress === null) {
    workInProgress = createFiber(current.tag, current.key, pendingProps);
    workInProgress.type = current.type;
    workInProgress.stateNode = current.stateNode;
    workInProgress.alternate = current;
    current.alternate = workInProgress;
  } else {
    workInProgress.pendingProps = pendingProps;
  }

  workInProgress.memoizedState = current.memoizedState;
  return workInProgress;
}
```

它做两种情况。

第一种：还没有备用 Fiber。

```txt
current.alternate === null
```

说明这是第一次为这个节点创建另一份 Fiber，于是：

```txt
创建 workInProgress
复制 type、stateNode
建立 alternate 双向连接
```

第二种：已经有备用 Fiber。

```txt
current.alternate !== null
```

说明之前已经创建过另一份 Fiber，这次直接复用它，只更新：

```ts
workInProgress.pendingProps = pendingProps;
```

最后无论哪种情况，都会复制：

```ts
workInProgress.memoizedState = current.memoizedState;
```

这是 hooks 状态能在更新时继续存在的重要原因。

---

## 4. 首次渲染时发生了什么

入口：

```ts
root.render(<App />);
```

会进入：

```ts
export function updateContainer(element: ReactElement, fiberRoot: FiberRoot) {
  fiberRoot.current!.memoizedState.element = element;
  updateOnFiber(fiberRoot);
}
```

先把新的 ReactElement 存到当前 `HostRootFiber` 上：

```txt
HostRootFiber.memoizedState.element -> <App />
```

然后进入更新流程：

```ts
export function updateOnFiber(fiberRoot: FiberRoot) {
  workInProgress = createWorkInProgress(
    fiberRoot.current!,
    fiberRoot.current!.pendingProps
  );

  workLoop();
  commitMutaionEffects(fiberRoot.current!.alternate!);
  fiberRoot.current = fiberRoot.current!.alternate;
}
```

首次渲染时，`fiberRoot.current` 是原始的 `HostRootFiber`。

`createWorkInProgress` 会创建它的 alternate：

```txt
current HostRootFiber
  alternate -> workInProgress HostRootFiber

workInProgress HostRootFiber
  alternate -> current HostRootFiber
```

然后从 `workInProgress HostRootFiber` 开始执行 `workLoop`。

---

## 5. 为什么 HostRoot 可以开始创建子节点

在 `beginWork` 里：

```ts
case HostRoot:
  fiber.child = reconcileChildFibers(fiber, fiber.memoizedState.element);
  return fiber.child;
```

因为前面已经把 element 存到了：

```txt
HostRootFiber.memoizedState.element
```

所以 HostRoot 的 `beginWork` 可以根据这个 element 创建子 Fiber。

比如：

```tsx
root.render(<App />);
```

会形成：

```txt
workInProgress HostRootFiber
  child -> App Fiber
```

后面再通过 `beginWork`、`completeWork` 继续往下创建完整 Fiber 树和 DOM。

---

## 6. 提交后切换 current

render 阶段结束后，新的 Fiber 树已经算好了。

当前项目里提交逻辑是：

```ts
commitMutaionEffects(fiberRoot.current!.alternate!);
fiberRoot.current = fiberRoot.current!.alternate;
```

第一句：

```ts
commitMutaionEffects(fiberRoot.current!.alternate!);
```

把 workInProgress 树对应的 DOM 挂到页面上。

第二句：

```ts
fiberRoot.current = fiberRoot.current!.alternate;
```

把 `FiberRoot.current` 指向刚刚完成的 workInProgress 树。

也就是：

```txt
提交前：
FiberRoot.current -> 旧树

提交后：
FiberRoot.current -> 新树
```

这一步就是双缓存切换的核心。

---

## 7. 更新时如何复用旧节点

更新时最关键的是：

```txt
新树不是完全凭空创建
它会借助 current 树上的 alternate 和旧 child 做对比
```

单节点协调在 `ChildFiber.ts`：

```ts
function reconcileSingleElement(returnFiber: any, children: any): Fiber {
  const key = children.key;
  let child = returnFiber.alternate?.child;

  while (child) {
    if (child.key === key) {
      const elementType = children.type;

      if (elementType === child.type) {
        const existing = createWorkInProgress(child, children.props);
        existing.return = returnFiber;
        deleteRemainingChildren(returnFiber, child.sibling);
        return existing;
      } else {
        deleteRemainingChildren(returnFiber, child);
        break;
      }
    } else {
      deleteChild(returnFiber, child);
    }

    child = child.sibling;
  }

  const created = createFiberFromElement(children);
  created.return = returnFiber;
  created.flags |= Placement;
  return created;
}
```

这里的关键是：

```ts
let child = returnFiber.alternate?.child;
```

`returnFiber` 是新树上的父 Fiber。

`returnFiber.alternate` 是旧树上的父 Fiber。

所以：

```txt
returnFiber.alternate.child
```

就是旧树上的第一个子 Fiber。

有了旧子 Fiber，才能进行对比：

```txt
key 相同？
type 相同？
```

如果都相同，就复用旧 Fiber 创建 workInProgress：

```ts
const existing = createWorkInProgress(child, children.props);
```

如果不同，就标记删除旧节点，并创建新节点。

---

## 8. Placement 和删除标记

当前项目里有两个和更新相关的 flags：

```ts
Placement
ChildDeletion
```

新创建的 Fiber 会打上：

```ts
created.flags |= Placement;
```

表示这个节点需要插入页面。

需要删除旧节点时，会执行：

```ts
returnFiber.flags |= ChildDeletion;
returnFiber.deletions = [childToDelete];
```

表示这个父 Fiber 下面有子节点需要删除。

不过要注意：

> 当前教学版已经记录了 Placement 和 ChildDeletion，但 commit 阶段还没有完整消费这些 flags。

现在的 `updateOnFiber` 仍然使用比较粗的方式：

```ts
if (fiberRoot.current?.child?.stateNode) {
  removeChild(fiberRoot.containerInfo, fiberRoot.current.child?.stateNode);
}
```

也就是先移除旧的根 DOM，再提交新的根 DOM。

这能帮助你理解更新闭环，但还不是 React 真正的细粒度 DOM 更新。

---

## 9. useState 和双缓存的关系

`useState` 更新时会进入：

```ts
function dispatchSetState(fiber: Fiber, hook: Hook, newState: any) {
  hook.memoizedState = newState;
  const fiberRoot = getRootForUpdateFiber(fiber);
  updateOnFiber(fiberRoot);
}
```

这里做了两件事：

第一，更新 hook 上保存的状态：

```ts
hook.memoizedState = newState;
```

第二，从当前 Fiber 向上找到 `FiberRoot`：

```ts
const fiberRoot = getRootForUpdateFiber(fiber);
```

然后重新调度：

```ts
updateOnFiber(fiberRoot);
```

在 `createWorkInProgress` 里又会复制：

```ts
workInProgress.memoizedState = current.memoizedState;
```

所以函数组件再次执行时，`renderWihtHooks` 能判断：

```ts
if (currentlyRenderingFiber!.memoizedState === null) {
  ReactSharedInternals.H = mountState;
} else {
  ReactSharedInternals.H = updateState;
}
```

只要 `memoizedState` 不为空，就说明不是首次 mount，而是 update。

于是：

```txt
mountState  -> 创建 hook
updateState -> 读取已有 hook
```

这也是 hook 状态能跨更新保留下来的原因。

---

## 10. 一次状态更新的完整流程

假设组件是：

```tsx
function App() {
  const [count, setCount] = useState(0);

  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

点击按钮后：

```txt
onClick
  ↓
setCount(count + 1)
  ↓
dispatchSetState
  ↓
hook.memoizedState = newState
  ↓
getRootForUpdateFiber
  ↓
updateOnFiber(fiberRoot)
  ↓
createWorkInProgress(fiberRoot.current)
  ↓
workLoop render 新树
  ↓
commitMutaionEffects
  ↓
fiberRoot.current = fiberRoot.current.alternate
```

简化成一句话：

> setState 改 hook 状态，然后基于 current 树创建 workInProgress 树，render 完成后提交并切换 current。

---

## 11. 一张图理解双缓存

第一次更新前：

```txt
FiberRoot
  current
    ↓
  HostRootFiber A
    alternate -> null
```

创建 workInProgress 后：

```txt
FiberRoot
  current
    ↓
  HostRootFiber A  <---- alternate ---->  HostRootFiber B
```

render 阶段在 B 上工作：

```txt
HostRootFiber B
  child -> App Fiber B
            child -> div/button Fiber B
```

commit 之后：

```txt
FiberRoot
  current
    ↓
  HostRootFiber B

HostRootFiber B.alternate -> HostRootFiber A
HostRootFiber A.alternate -> HostRootFiber B
```

下一次更新时，又会从 B 的 alternate A 开始复用。

所以两棵树会来回切换：

```txt
第 1 次提交：A -> B
第 2 次提交：B -> A
第 3 次提交：A -> B
```

这就是双缓存。

---

## 12. 容易混淆的点

### `FiberRoot.current` 不是“正在渲染的 Fiber”

`FiberRoot.current` 表示：

```txt
当前已经提交、生效、对应页面的 Fiber 树
```

正在渲染的 Fiber 保存在全局变量：

```ts
let workInProgress: Fiber | null = null;
```

它表示当前 workLoop 正在处理的工作单元。

---

### `workInProgress` 有两层含义

项目里有一个变量：

```ts
let workInProgress: Fiber | null = null;
```

它表示：

```txt
当前正在处理的 Fiber 节点
```

而我们平时说的 `workInProgress tree`，表示：

```txt
整棵正在构建的新 Fiber 树
```

一个是单个指针，一个是一整棵树。

---

### `alternate` 不是父子关系

父子关系用：

```txt
return / child / sibling
```

双缓存关系用：

```txt
alternate
```

所以：

```txt
fiber.child      -> 子 Fiber
fiber.sibling    -> 兄弟 Fiber
fiber.return     -> 父 Fiber
fiber.alternate  -> 另一棵树上对应的 Fiber
```

---

### `pendingProps` 和 `memoizedState`

在当前项目里可以先这样理解：

```txt
pendingProps   -> 这次 render 要使用的新 props
memoizedState  -> 上一次保存下来的状态
```

对 HostRootFiber 来说：

```txt
memoizedState.element -> root.render 传进来的 ReactElement
```

对函数组件 Fiber 来说：

```txt
memoizedState -> hook 链表的第一个 hook
```

---

## 13. 当前实现和真实 React 的差异

当前项目是教学版，重点是把主线跑通，所以有一些简化。

第一，commit 阶段还不是细粒度更新。

当前做法是：

```txt
移除旧根 DOM
挂载新根 DOM
```

真实 React 会根据 flags 精确处理插入、删除、更新。

第二，数组 children 的更新对比还比较简单。

当前数组逻辑主要是创建 Fiber 链表，还没有完整实现 key diff。

第三，更新队列还没有完整实现。

当前 `setState` 是直接改 hook 的 `memoizedState`，真实 React 会把更新放进 update queue，再在 render 阶段计算新状态。

这些简化不影响你先理解双缓存主线：

```txt
current 树
  ↓ createWorkInProgress
workInProgress 树
  ↓ render
commit
  ↓
切换 FiberRoot.current
```

---

## 14. 复习自测

### 1. 为什么需要 `alternate`？

为了让同一个节点在 current 树和 workInProgress 树之间建立联系。

有了它，更新时才能从旧 Fiber 读取状态、DOM、旧子节点，并创建或复用新的 Fiber。

---

### 2. `FiberRoot.current` 指向谁？

指向当前已经提交的 `HostRootFiber`。

不是 `FiberRoot` 自己，也不是普通组件 Fiber。

---

### 3. `createWorkInProgress` 什么时候创建新 Fiber，什么时候复用旧 Fiber？

如果：

```txt
current.alternate === null
```

就创建新的 alternate Fiber。

如果：

```txt
current.alternate !== null
```

就复用已有 alternate，只更新 `pendingProps` 等数据。

---

### 4. 提交后为什么要执行 `fiberRoot.current = fiberRoot.current.alternate`？

因为刚刚 render 完成的是 alternate 那棵树。

commit 后它已经代表最新页面，所以要把它切换成新的 current 树。

---

### 5. 更新时怎么找到旧子节点？

在新父 Fiber 上通过：

```ts
returnFiber.alternate?.child
```

找到旧父 Fiber 的旧子节点。

然后根据 `key` 和 `type` 判断能不能复用。

---

## 15. 本讲小结

这一讲最重要的闭环是：

```txt
FiberRoot.current 指向当前树
  ↓
createWorkInProgress 创建/复用 alternate 树
  ↓
workLoop 在 workInProgress 树上 render
  ↓
commit 把结果放到页面上
  ↓
FiberRoot.current 切换到 workInProgress 树
```

只要记住这句话，双缓存的主干就稳了：

> current 是已经生效的树，alternate 是另一份可复用的树，更新时在 alternate 上工作，提交后 alternate 变成新的 current。

