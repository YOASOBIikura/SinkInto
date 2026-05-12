---
title: React-03
date: 2026-05-12 11:00:00
categories:
  - ReactCore
tags:
  - HostRootFiber创建
---

# 第 2 讲：createRoot 和 root.render

核心文件：

- `packages/react-dom/client.ts`
- `packages/reconciler/FiberReconciler.ts`

这一讲要回答的问题是：

> ReactElement 创建出来以后，是怎么进入渲染流程的？

上一讲我们知道 JSX 会变成 ReactElement：

```tsx
const element = <div id="app">hello</div>
```

本质是：

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

但这个对象本身不会自动出现在页面上。它需要通过：

```ts
createRoot(container).render(element)
```

送进 reconciler。

---

## 1. `createRoot` 做了什么

源码：

```ts
function createRoot(container: HTMLElement) {
  const hostRootFiber = createContainer(container);
  return ReactDomRoot(hostRootFiber);
}
```

`createRoot` 接收真实 DOM 容器，比如：

```ts
const root = createRoot(document.getElementById("root"));
```

这里的 `container` 就是页面里真实存在的 DOM 节点。

然后它调用：

```ts
createContainer(container)
```

这个函数来自：

```txt
packages/reconciler/FiberReconciler.ts
```

它会创建 React 内部的根节点，也就是 `HostRootFiber`。

---

## 2. `ReactDomRoot` 是什么

源码：

```ts
function ReactDomRoot(hostRootFiber: Fiber): ReactDomRootType {
  return {
    _internalRoot: hostRootFiber,
    render: function (element: ReactElement) {
      updateContainer(element, this._internalRoot);
    }
  };
}
```

`createRoot` 返回的不是 DOM，也不是 Fiber，而是一个包装对象：

```ts
{
  _internalRoot: hostRootFiber,
  render(element) {
    updateContainer(element, this._internalRoot)
  }
}
```

所以当你写：

```ts
root.render(<App />);
```

实际调用的是：

```ts
updateContainer(<App />, hostRootFiber);
```

这一步非常关键：

> `root.render(element)` 的作用，就是把 ReactElement 交给 reconciler。

---

## 3. `createContainer` 做了什么

源码：

```ts
export function createContainer(containerInfo: HTMLElement) {
  const root = createFiberRoot(containerInfo);
  const hostRootFiber = createHostRootFiber();
  hostRootFiber.stateNode = root;
  listenToAllSupportedEvents(root.containerInfo);
  return hostRootFiber
}
```

这里做了几件事。

第一，创建 `FiberRoot`：

```ts
const root = createFiberRoot(containerInfo);
```

`FiberRoot` 保存真实 DOM 容器：

```ts
{
  containerInfo: HTMLElement
}
```

第二，创建 `HostRootFiber`：

```ts
const hostRootFiber = createHostRootFiber();
```

`HostRootFiber` 是 React 内部 Fiber 树的根节点。

第三，把它们关联起来：

```ts
hostRootFiber.stateNode = root;
```

也就是说：

```txt
HostRootFiber.stateNode -> FiberRoot
FiberRoot.containerInfo -> 真实 DOM 容器
```

第四，注册事件系统：

```ts
listenToAllSupportedEvents(root.containerInfo);
```

这一步会在根容器上监听 `click`、`focus`、`input` 等事件，为后面的合成事件系统做准备。

---

## 4. `updateContainer` 做了什么

源码：

```ts
export function updateContainer(element: ReactElement, root: Fiber) {
  const containerFiber = createFiberFromElement(element);
  workLoop(containerFiber);

  root.child = containerFiber;
  containerFiber.return = root;

  commitMutaionEffects(root);
}
```

这里就是从 ReactElement 进入 Fiber 渲染流程的真正入口。

它分几步。

第一步，把 ReactElement 变成 Fiber：

```ts
const containerFiber = createFiberFromElement(element);
```

比如：

```tsx
<div id="app">hello</div>
```

会从 ReactElement 变成一个 `HostComponent Fiber`。

第二步，执行 render 阶段：

```ts
workLoop(containerFiber);
```

`workLoop` 会开始构建整棵 Fiber 树，并在回溯阶段创建真实 DOM。

第三步，把新 Fiber 接到根节点下面：

```ts
root.child = containerFiber;
containerFiber.return = root;
```

形成这样的结构：

```txt
HostRootFiber
  child -> App 或 div Fiber
```

第四步，进入 commit 阶段：

```ts
commitMutaionEffects(root);
```

也就是把真实 DOM 插入页面容器。

---

## 5. `FiberRoot` 和 `HostRootFiber` 的区别

这两个名字很像，但职责不同。

`FiberRoot` 是整个 React 应用的根对象，它保存真实 DOM 容器：

```txt
FiberRoot.containerInfo -> 真实 DOM 容器
```

`HostRootFiber` 是 Fiber 树的根 Fiber 节点，它负责接住真正的应用 Fiber：

```txt
HostRootFiber.child -> App Fiber 或 div Fiber
```

它们通过 `stateNode` 关联：

```txt
HostRootFiber.stateNode -> FiberRoot
```

可以把它理解成：

```txt
真实 DOM 容器
  ↑
FiberRoot
  ↑
HostRootFiber
  ↓
应用 Fiber 树
```

---

## 6. 本章流程图

```txt
createRoot(container)
  ↓
createContainer(container)
  ↓
创建 FiberRoot
  ↓
创建 HostRootFiber
  ↓
HostRootFiber.stateNode = FiberRoot
  ↓
返回 ReactDomRoot 对象
  ↓
root.render(element)
  ↓
updateContainer(element, HostRootFiber)
  ↓
ReactElement -> Fiber
  ↓
workLoop 构建 Fiber/DOM
  ↓
commitMutaionEffects 挂载到页面
```

---

## 7. 本章总结

这一章最重要的一句话：

> `createRoot` 创建 React 应用的内部根节点，`root.render` 把 ReactElement 送进 reconciler，开始 Fiber 构建和 DOM 挂载。

可以更具体地拆成三句话：

- `createRoot(container)` 保存真实 DOM 容器，并创建 React 内部根节点。
- `root.render(element)` 把 ReactElement 交给 `updateContainer`。
- `updateContainer` 负责把 ReactElement 转成 Fiber，然后启动 render 和 commit 流程。

从整体链路看：

```txt
ReactElement
  ↓
root.render
  ↓
updateContainer
  ↓
Fiber
  ↓
DOM
```

下一章可以继续看 **Fiber 数据结构**。
