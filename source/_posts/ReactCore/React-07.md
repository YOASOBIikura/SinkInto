---
title: React-06
date: 2026-05-12 15:08:33
categories:
  - ReactCore
tags:
  - DOM Binding 和事件系统
---

# 第 7 讲：DOM Binding 和事件系统

核心文件：

- `packages/react-dom-binding/FiberConfigDOM.ts`
- `packages/react-dom-binding/ReactDomComponentTree.ts`
- `packages/react-dom-binding/DomPluginEventSystem.ts`
- `packages/react-dom-binding/SyntheticeEvent.ts`
- `packages/reconciler/FiberReconciler.ts`

这一讲要回答两个问题：

> reconciler 是怎么操作真实 DOM 的？

> React 事件是怎么从浏览器原生事件，变成组件里的 `onClick` 调用的？

前面几章里我们已经看到：

```txt
ReactElement
  ↓
Fiber
  ↓
workLoop
  ↓
completeWork 创建 DOM
  ↓
commit 阶段挂载 DOM
```

这一章看的就是最靠近浏览器的那一层：

```txt
react-dom-binding
```

---

## 1. 什么是 DOM Binding

可以先用一句话理解：

> DOM Binding 是 reconciler 和浏览器 DOM 之间的适配层。

reconciler 负责通用流程：

```txt
创建 Fiber
遍历 Fiber
创建节点
挂载节点
更新节点
```

但真正的宿主环境可能不同。

如果目标是浏览器，需要调用：

```ts
document.createElement(...)
parent.appendChild(...)
dom.setAttribute(...)
```

如果目标是其它平台，比如 canvas、小程序、终端 UI，这些操作就不是浏览器 DOM API 了。

所以项目里把浏览器相关操作放在：

```txt
packages/react-dom-binding/FiberConfigDOM.ts
```

这样 reconciler 就通过这些函数操作宿主环境。

---

## 2. 创建 DOM 节点

源码：

```ts
export function createDom(type: string, fiber: Fiber) {
  let domElement = document.createElement(type);
  precacheFiberNode(fiber, domElement);
  return document.createElement(type);
}
```

这个函数的职责是：

```txt
根据 Fiber.type 创建真实 DOM
```

比如：

```tsx
<div />
```

对应的 `fiber.type` 是：

```ts
"div"
```

所以会调用：

```ts
document.createElement("div")
```

创建出来的 DOM 会在 `completeWork` 里保存到：

```ts
fiber.stateNode
```

也就是：

```txt
div Fiber.stateNode -> div DOM
```

---

## 3. 这里有一个关键小问题

当前 `createDom` 代码里创建了两个 DOM：

```ts
let domElement = document.createElement(type);
precacheFiberNode(fiber, domElement);
return document.createElement(type);
```

`precacheFiberNode` 绑定的是第一个 DOM。

但真正返回并挂到页面上的是第二个 DOM。

这会导致：

```txt
页面上的真实 DOM 没有绑定 Fiber
```

事件系统依赖从 DOM 找回 Fiber，所以这里会影响事件派发。

更合理的写法应该是：

```ts
export function createDom(type: string, fiber: Fiber) {
  const domElement = document.createElement(type);
  precacheFiberNode(fiber, domElement);
  return domElement;
}
```

这一点非常重要，因为后面的事件系统需要：

```txt
event.target DOM -> 对应 Fiber
```

---

## 4. 创建文本节点

源码：

```ts
export function createTextInstance(text: string) {
  return document.createTextNode(text);
}
```

文本 Fiber 在 `completeWork` 阶段会调用它：

```ts
fiber.stateNode = createTextInstance(fiber.pendingProps);
```

形成：

```txt
HostText Fiber.stateNode -> Text
```

---

## 5. 插入和删除 DOM

源码：

```ts
export function appendChild(parent: Instance, child: Instance) {
  parent.appendChild(child);
}

export function removeChild(parent: Instance, child: Instance) {
  parent.removeChild(child);
}
```

`appendChild` 会在两个地方用到。

第一，render 阶段的 `completeWork`：

```ts
appendAllChildren(instance, fiber.child);
```

把子 DOM 挂到父 DOM 上。

第二，commit 阶段：

```ts
appendChild(fiber.stateNode.containerInfo, fiber.child?.stateNode);
```

把应用根 DOM 挂到页面容器上。

`removeChild` 会在更新时用到：

```ts
removeChild(hostRootFiber.stateNode.containerInfo, hostRootFiber.child?.stateNode);
```

当前教学版更新策略是先删旧 DOM，再重新 render 和 commit。

---

## 6. 设置初始属性

源码：

```ts
export function setInitialProps(dom: Instance, props: any) {
  for (const prop in props) {
    if (!props.hasOwnProperty(prop)) {
      continue;
    }

    if (prop === 'children') {
      if (typeof props.children === 'string' || typeof props.children === 'number') {
        dom.textContent = props.children;
      }
      continue;
    }

    dom.setAttribute(prop, props[prop])
  }
}
```

这个函数负责把 Fiber 的 props 设置到真实 DOM 上。

比如：

```tsx
<div id="app">hello</div>
```

对应：

```ts
props = {
  id: "app",
  children: "hello"
}
```

会执行：

```ts
dom.setAttribute("id", "app")
dom.textContent = "hello"
```

注意：

```ts
children
```

不是作为普通属性设置的。

如果 children 是字符串或数字，会变成：

```ts
dom.textContent
```

---

## 7. 当前属性设置的局限

当前 `setInitialProps` 会对非 children 属性统一调用：

```ts
dom.setAttribute(prop, props[prop])
```

这对普通属性可以工作，比如：

```tsx
<div id="app" title="hello" />
```

但对事件属性不合适：

```tsx
<button onClick={handleClick}>click</button>
```

当前代码会尝试：

```ts
button.setAttribute("onClick", handleClick)
```

真实 React 不会把 `onClick` 当普通 DOM 属性直接 setAttribute。

真实 React 会把事件交给事件系统处理。

你的教学版事件系统也是通过 Fiber 的 `pendingProps.onClick` 读取监听函数，而不是通过 DOM 属性触发。

---

## 8. DOM 和 Fiber 的绑定

核心文件：

```txt
packages/react-dom-binding/ReactDomComponentTree.ts
```

源码：

```ts
let randomKey = Math.random().toString(36).slice(2);
export let internalInstanceKey = '__reactFiber$' + randomKey;

export function precacheFiberNode(fiber: Fiber, instance: Instance) {
  (instance as any)[internalInstanceKey] = fiber;
}
```

这里做了一件很关键的事：

> 在真实 DOM 节点上挂一个隐藏属性，指向对应的 Fiber。

大概像这样：

```ts
dom.__reactFiber$abc123 = fiber
```

于是后面事件发生时，就可以从：

```txt
event.target DOM
```

反查到：

```txt
target Fiber
```

这就是事件系统能工作的基础。

---

## 9. 事件系统在哪里注册

在创建根容器时：

```ts
export function createContainer(containerInfo: HTMLElement) {
  const root = createFiberRoot(containerInfo);
  const hostRootFiber = createHostRootFiber();
  hostRootFiber.stateNode = root;
  listenToAllSupportedEvents(root.containerInfo);
  return hostRootFiber
}
```

这里调用：

```ts
listenToAllSupportedEvents(root.containerInfo);
```

也就是说：

> React 会在根容器上统一注册事件监听。

它不是给每一个按钮都单独调用：

```ts
button.addEventListener("click", ...)
```

而是在根容器上监听：

```ts
rootContainer.addEventListener("click", ...)
```

这就是事件委托。

---

## 10. 原生事件名到 React 事件名的映射

源码：

```ts
const topLevelEventsToReactNames: Map<string, string> = new Map([
  ['click', 'onClick'],
  ['focus', 'onFocus'],
  ['input', 'onInput']
])
```

浏览器原生事件名是：

```txt
click
focus
input
```

React props 里的事件名是：

```txt
onClick
onFocus
onInput
```

所以事件系统需要一张映射表。

当浏览器触发 `click` 时，React 要去 Fiber 的 props 里找：

```ts
pendingProps.onClick
```

---

## 11. 注册根事件监听

源码：

```ts
export function listenToAllSupportedEvents(rootContainerElement: EventTarget) {
  topLevelEventsToReactNames.forEach((nativeEvent, reactName) => {
    rootContainerElement.addEventListener(nativeEvent, (e) => {
      const listeners = accumulateSinglePhaseListeners(
        (e.target as any)[internalInstanceKey],
        reactName
      );
      const syntheticEvent = createSyntheticEvent(e);
      processEventQueueItemsInOrder(syntheticEvent, listeners);
    })
  })
}
```

这里的流程是：

```txt
在根容器监听原生事件
  ↓
事件触发后拿到 e.target
  ↓
通过 internalInstanceKey 找到 target Fiber
  ↓
从 target Fiber 向上收集 onClick/onInput 等监听函数
  ↓
创建合成事件
  ↓
按顺序执行 listener
```

---

## 12. 从 DOM 找到 Fiber

关键代码：

```ts
(e.target as any)[internalInstanceKey]
```

因为创建 DOM 时调用了：

```ts
precacheFiberNode(fiber, domElement);
```

所以真实 DOM 上应该有：

```ts
dom[internalInstanceKey] = fiber
```

这样点击某个 DOM 时：

```txt
e.target
  ↓
e.target[internalInstanceKey]
  ↓
target Fiber
```

再次提醒：当前 `createDom` 返回了另一个 DOM，这会导致实际挂载的 DOM 上没有这个绑定。事件系统想正常工作，需要修成返回已绑定的 `domElement`。

---

## 13. 收集事件监听器

源码：

```ts
export function accumulateSinglePhaseListeners(
  targetFiber: Fiber,
  reactName: string
): Array<any> {
  let fiber: Fiber | null = targetFiber;
  const listeners: Array<DispatchListener> = [];

  while (fiber) {
    const { pendingProps, tag } = fiber;
    if (tag === HostComponent) {
      const lisener = pendingProps[reactName];
      if (typeof lisener === 'function') {
        listeners.push(createDispatchListener(fiber, lisener, fiber.stateNode));
      }
    }
    fiber = fiber.return;
  }

  return listeners;
}
```

它从目标 Fiber 开始，沿着：

```txt
fiber.return
```

一路向上找父 Fiber。

如果某个 Fiber 是 `HostComponent`，并且它的 props 里有对应事件：

```ts
pendingProps[reactName]
```

就收集起来。

比如：

```tsx
<div onClick={handleDivClick}>
  <button onClick={handleButtonClick}>click</button>
</div>
```

点击 button 时，会从 `button Fiber` 开始向上找：

```txt
button Fiber -> div Fiber -> HostRootFiber
```

收集到：

```txt
button onClick
div onClick
```

这就是模拟事件冒泡链。

---

## 14. DispatchListener

源码：

```ts
type DispatchListener = {
  listener: Function;
  currentTarget: EventTarget | null;
  fiber: Fiber | null;
}
```

每个收集到的 listener 会被包装成：

```ts
{
  listener,
  currentTarget,
  fiber
}
```

其中：

```ts
currentTarget
```

表示当前正在触发 listener 的那个 DOM。

比如 button 的 listener 执行时：

```txt
event.currentTarget -> button DOM
```

div 的 listener 执行时：

```txt
event.currentTarget -> div DOM
```

---

## 15. 合成事件 SyntheticEvent

核心文件：

```txt
packages/react-dom-binding/SyntheticeEvent.ts
```

源码：

```ts
type SyntheticEvent = {
  nativeEvent: Event;
  currentTarget: EventTarget | null;
  stopPropagation: () => void;
  isPropagationStopped: () => boolean;
}
```

合成事件保存了：

```txt
nativeEvent              -> 浏览器原生事件
currentTarget            -> 当前正在执行 listener 的 DOM
stopPropagation          -> 停止传播
isPropagationStopped     -> 是否已经停止传播
```

创建合成事件：

```ts
export default function createSyntheticEvent(nativeEvent: Event) {
  return new SyntheticEvent(nativeEvent);
}
```

也就是说，浏览器给的是原生事件：

```txt
MouseEvent
```

React 事件系统会包装成：

```txt
SyntheticEvent
```

再传给组件里的事件处理函数。

---

## 16. `stopPropagation`

源码：

```ts
SyntheticEvent.prototype = {
  stopPropagation: function() {
    this.isPropagationStopped = functionThatReturnTrue;
  },
  isPropagationStopped: functionThatReturnFalse
}
```

默认情况下：

```ts
event.isPropagationStopped()
```

返回：

```ts
false
```

调用：

```ts
event.stopPropagation()
```

之后，会把：

```ts
isPropagationStopped
```

改成返回 `true` 的函数。

这是一种很轻量的状态切换方式。

---

## 17. 执行事件队列

源码：

```ts
export function processEventQueueItemsInOrder(event: any, listeners: Array<any>) {
  for (let i = 0; i < listeners.length; i++) {
    const { listener, currentTarget, fiber } = listeners[i];
    event.currentTarget = currentTarget;
    listener(event);
    event.currentTarget = null;
    if (event.isPropagationStopped()) {
      return;
    }
  }
}
```

执行过程是：

```txt
取出 listener
  ↓
设置 event.currentTarget
  ↓
调用 listener(event)
  ↓
清空 event.currentTarget
  ↓
如果 stopPropagation 了，就停止后续 listener
```

比如：

```tsx
<div onClick={onDivClick}>
  <button
    onClick={(e) => {
      e.stopPropagation();
    }}
  >
    click
  </button>
</div>
```

点击 button 时：

```txt
执行 button listener
  ↓
调用 stopPropagation
  ↓
isPropagationStopped() 返回 true
  ↓
停止
  ↓
div listener 不执行
```

---

## 18. 完整事件流程

假设 JSX 是：

```tsx
<div onClick={handleDivClick}>
  <button onClick={handleButtonClick}>click</button>
</div>
```

点击 button 后，完整流程是：

```txt
浏览器触发 click
  ↓
事件冒泡到 rootContainer
  ↓
rootContainer 上的 click listener 执行
  ↓
通过 e.target[internalInstanceKey] 找到 button Fiber
  ↓
把 click 映射成 onClick
  ↓
从 button Fiber 向上收集 pendingProps.onClick
  ↓
得到 [button listener, div listener]
  ↓
createSyntheticEvent(e)
  ↓
依次执行 listener
  ↓
每次执行前设置 event.currentTarget
  ↓
如果 stopPropagation，停止后续 listener
```

---

## 19. 当前实现和真实 React 的差异

当前教学版已经实现了事件系统的核心骨架：

- 根容器统一监听事件
- 原生事件名映射到 React 事件名
- DOM 反查 Fiber
- 沿 Fiber.return 收集冒泡链 listener
- 创建 SyntheticEvent
- 支持 `currentTarget`
- 支持 `stopPropagation`

但它还不是完整 React 事件系统。

当前没有处理：

- 捕获阶段，比如 `onClickCapture`
- 事件优先级
- 离散事件和连续事件
- 事件插件扩展
- 不同事件的特殊冒泡规则
- `preventDefault`
- 事件对象属性代理
- 事件更新批处理
- hydration 相关事件重放

另外当前 DOM 创建处的小问题会影响事件系统：

```ts
precacheFiberNode(fiber, domElement);
return document.createElement(type);
```

如果不修，真实挂载的 DOM 上没有 Fiber 绑定，事件发生时可能无法通过 `e.target[internalInstanceKey]` 找到 Fiber。

---

## 20. 本章总结

这一章最重要的一句话：

> DOM Binding 负责操作真实 DOM，事件系统通过根容器事件委托、DOM 反查 Fiber、沿 Fiber.return 收集 listener，再用 SyntheticEvent 统一派发。

可以拆成几句：

- `FiberConfigDOM` 是浏览器 DOM 的 host config。
- `createDom` 创建真实 DOM，并应该把 DOM 和 Fiber 绑定。
- `setInitialProps` 把 props 设置到 DOM 上。
- `createContainer` 时会在根容器注册支持的事件。
- 事件触发后，通过 `event.target[internalInstanceKey]` 找到目标 Fiber。
- `accumulateSinglePhaseListeners` 沿 `return` 指针向上收集事件处理函数。
- `createSyntheticEvent` 把原生事件包装成合成事件。
- `processEventQueueItemsInOrder` 按冒泡顺序执行 listener。

整体流程是：

```txt
创建 DOM
  ↓
DOM 绑定 Fiber
  ↓
根容器注册事件
  ↓
浏览器事件触发
  ↓
event.target DOM
  ↓
通过 internalInstanceKey 找到 Fiber
  ↓
沿 Fiber.return 收集 listener
  ↓
创建 SyntheticEvent
  ↓
按顺序执行 listener
```

到这里，这个教学版 React 的主线已经基本串起来了：

```txt
JSX
  ↓
ReactElement
  ↓
Fiber
  ↓
render 阶段
  ↓
commit 阶段
  ↓
DOM Binding
  ↓
事件系统
```


