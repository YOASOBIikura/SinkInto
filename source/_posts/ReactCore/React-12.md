---
title: React-08
date: 2026-06-01 17:24:59
categories:
  - ReactCore
tags:
  - Update 标记
---

# 第 13 讲：Update 标记

核心文件：

- `packages/reconciler/CompleteWork.ts`
- `packages/reconciler/CommitWork.ts`
- `packages/reconciler/FiberFlags.ts`
- `packages/react-dom-binding/FiberConfigDOM.ts`

这一讲要回答的问题是：

> React 发现 DOM 属性或文本内容变化之后，是怎么打 Update 标记，并在 commit 阶段更新真实 DOM 的？

前面我们已经复习了：

```txt
Placement       插入或移动
ChildDeletion   删除子节点
```

这两个主要和 DOM 结构有关。

`Update` 不一样。

它主要表示：

```txt
节点位置不一定变，但节点内容或属性变了
```

比如：

```tsx
<div id="old">hello</div>
```

更新为：

```tsx
<div id="new">hello</div>
```

这个 `div` 可以复用，不需要重新插入，也不需要删除。

但它的属性变了，所以需要打：

```txt
Update
```

---

## 1. Update 是什么

源码：

```ts
export type Flags = number;

export const NoFlags = 0b0000000;

export const Placement = 0b0000010;
export const Update = 0b0000100;
export const ChildDeletion = 0b0001000;

export const MutationMask = Placement | Update | ChildDeletion;
```

`Update` 是一个 mutation flag。

它表示：

```txt
当前 Fiber 对应的真实 DOM 需要更新
```

常见更新包括：

```txt
HostComponent   更新 DOM props
HostText        更新文本内容
```

---

## 2. Update 打在哪里

`Update` 打在当前 Fiber 自己身上。

比如：

```txt
div props 变了
```

那就是：

```txt
div Fiber.flags |= Update
```

比如：

```txt
文本内容变了
```

那就是：

```txt
HostText Fiber.flags |= Update
```

和 `Placement` 类似，`Update` 对应的 Fiber 会出现在新的 workInProgress 树里。

所以 commit 阶段遍历新树时，可以直接处理当前 Fiber。

---

## 3. Update 在哪里产生

`Update` 不是在 `ChildFiber.ts` 的 children diff 里主要产生的。

它主要在 `completeWork` 阶段产生。

入口：

```ts
export function completeWork(workInProgress: Fiber) {
  const current = workInProgress.alternate || null;

  const oldProps = current?.pendingProps || null;
  const newProps = workInProgress.pendingProps;

  switch (workInProgress.tag) {
    case HostComponent:
      if (current && workInProgress.stateNode !== null) {
        updateHostComponent(workInProgress, oldProps, newProps);
      } else {
        // mount 创建 DOM
      }
      bubbleProperties(workInProgress);
      break;

    case HostText:
      if (current && workInProgress.stateNode !== null) {
        updateHostText(workInProgress, oldProps, newProps);
      } else {
        // mount 创建文本 DOM
      }
      bubbleProperties(workInProgress);
      break;
  }
}
```

这里先判断：

```txt
current 存在
stateNode 存在
```

如果都存在，说明这是更新路径。

如果不存在，说明是首次挂载或新节点创建路径。

---

## 4. markUpdate：真正打标记

源码：

```ts
function markUpdate(workInProgress: Fiber) {
  workInProgress.flags |= Update;
}
```

这句就是：

```txt
给当前 Fiber 添加 Update 标记
```

因为 flags 是位运算，所以它不会覆盖其他标记。

比如一个 Fiber 也可以同时有：

```txt
Placement | Update
```

按位或之后：

```ts
workInProgress.flags |= Update;
```

只是在原来的 flags 上追加 `Update`。

---

## 5. HostComponent 的 Update

`HostComponent` 表示普通 DOM 元素。

比如：

```tsx
<div />
<span />
<button />
```

判断 props 是否变化的函数是：

```ts
function updateHostComponent(workInProgress: Fiber, oldProps: any, newProps: any) {
  if (oldProps !== newProps) {
    markUpdate(workInProgress);
  }
}
```

如果：

```txt
oldProps !== newProps
```

就打：

```txt
Update
```

这里的判断比较粗糙。

它不是逐个 prop 深度比较，而是直接比较引用。

对于教学版来说，重点是先理解：

```txt
completeWork 发现 props 变化
给 HostComponent Fiber 打 Update
commit 阶段再真正更新 DOM 属性
```

---

## 6. HostText 的 Update

`HostText` 表示文本节点。

比如：

```tsx
<div>hello</div>
```

里面的 `hello` 会对应一个 `HostText Fiber`。

判断文本是否变化的函数是：

```ts
function updateHostText(workInProgress: Fiber, oldProps: any, newProps: any) {
  if (oldProps !== newProps) {
    markUpdate(workInProgress);
  }
}
```

文本节点的 `pendingProps` 通常就是文本内容。

比如：

```txt
oldProps = "hello"
newProps = "world"
```

如果不同，就打：

```txt
HostText Fiber.flags |= Update
```

---

## 7. mount 时不会打 Update

在 `completeWork` 里，`HostComponent` 分为两条路径：

```ts
if (current && workInProgress.stateNode !== null) {
  updateHostComponent(workInProgress, oldProps, newProps);
} else {
  const instance = createDom(workInProgress.type, workInProgress);
  appendAllChildren(instance, workInProgress.child);
  setInitialProps(instance, workInProgress.pendingProps);
  workInProgress.stateNode = instance;
}
```

如果是 mount 或新节点：

```txt
current 不存在
或 stateNode 不存在
```

会直接创建 DOM，并设置初始属性：

```ts
setInitialProps(instance, workInProgress.pendingProps);
```

这时不需要打 `Update`。

因为 DOM 还没插入页面，初始属性可以直接设置好。

`Update` 主要用于：

```txt
已经存在的 DOM 需要改变
```

---

## 8. Update 和 createWorkInProgress 的关系

更新路径能够比较新旧 props，是因为 Fiber 之间有 `alternate`。

在 `completeWork` 里：

```ts
const current = workInProgress.alternate || null;
const oldProps = current?.pendingProps || null;
const newProps = workInProgress.pendingProps;
```

其中：

```txt
current       旧 Fiber
workInProgress 新 Fiber
oldProps      旧 props
newProps      新 props
```

如果 diff 阶段复用了旧 Fiber：

```ts
createWorkInProgress(oldFiber, newProps)
```

新旧 Fiber 就通过 `alternate` 连起来。

这样 complete 阶段才能知道：

```txt
旧 props 是什么
新 props 是什么
```

并决定是否打 `Update`。

---

## 9. bubbleProperties：让 Update 被父级知道

打在某个 Fiber 自己身上的 `Update`，还需要通过 `subtreeFlags` 往上冒泡。

源码：

```ts
function bubbleProperties(workInProgress: Fiber) {
  let subtreeFlags = NoFlags;

  let child = workInProgress.child;
  while (child) {
    subtreeFlags |= child.flags;
    subtreeFlags |= child.subtreeFlags;
    child = child.sibling;
  }

  workInProgress.subtreeFlags = subtreeFlags;
}
```

如果某个子节点有：

```txt
Update
```

它的父级 `subtreeFlags` 也会包含 `Update`。

这样 commit 阶段就可以通过：

```ts
finishedWork.subtreeFlags & MutationMask
```

判断要不要继续往子树里找副作用。

---

## 10. Commit 阶段如何消费 Update

入口在 `CommitWork.ts`：

```ts
function commitMutaionEffectsOnFiber(finishedWork: Fiber) {
  const flags = finishedWork.flags;

  recursivelyTraversMutationEffects(finishedWork);

  if (flags & Placement) {
    commitHostPlacement(finishedWork);
    finishedWork.flags &= ~Placement;
  }

  switch (finishedWork.tag) {
    case HostComponent:
      if (flags & Update) {
        const dom: Instance = finishedWork.stateNode;
        if (dom) {
          const newProps = finishedWork.pendingProps;
          const oldProps = finishedWork.alternate?.pendingProps;
          commitHostUpdate(dom, oldProps, newProps);
        }
      }
      break;

    case HostText:
      if (flags & Update) {
        const newText = finishedWork.pendingProps;
        commitTextUpdate(finishedWork.stateNode, newText);
      }
      break;
  }
}
```

可以看出：

```txt
HostComponent Update   -> commitHostUpdate
HostText Update        -> commitTextUpdate
```

---

## 11. HostComponent：commitHostUpdate

源码：

```ts
function commitHostUpdate(dom: Instance, oldProps: any, newProps: any) {
  for (const oldKey in oldProps) {
    const oldValue = oldProps[oldKey];

    if (oldProps.hasOwnProperty(oldKey) && oldValue !== null && !newProps.hasOwnProperty(oldKey)) {
      setProp(dom, oldKey, null);
    }
  }

  for (const newKey in newProps) {
    const newValue = newProps[newKey];
    const oldValue = oldProps[newKey];

    if (newProps.hasOwnProperty(newKey) && newValue !== oldValue && newValue !== null) {
      setProp(dom, newKey, newValue);
    }
  }
}
```

它分两步：

```txt
1. 删除新 props 里已经没有的旧属性
2. 设置新增或变化了的新属性
```

---

## 12. 删除旧属性

第一段循环：

```ts
for (const oldKey in oldProps) {
  const oldValue = oldProps[oldKey];

  if (oldProps.hasOwnProperty(oldKey) && oldValue !== null && !newProps.hasOwnProperty(oldKey)) {
    setProp(dom, oldKey, null);
  }
}
```

意思是：

```txt
旧 props 里有
新 props 里没有
```

那就删除这个属性。

例子：

旧：

```tsx
<div id="box" title="hello" />
```

新：

```tsx
<div id="box" />
```

`title` 旧有新无，所以应该清掉。

当前代码通过：

```ts
setProp(dom, oldKey, null);
```

表示删除或清空属性。

---

## 13. 设置新属性

第二段循环：

```ts
for (const newKey in newProps) {
  const newValue = newProps[newKey];
  const oldValue = oldProps[newKey];

  if (newProps.hasOwnProperty(newKey) && newValue !== oldValue && newValue !== null) {
    setProp(dom, newKey, newValue);
  }
}
```

意思是：

```txt
新 props 里有
并且新旧值不同
并且新值不是 null
```

那就更新这个属性。

例子：

旧：

```tsx
<div id="old" />
```

新：

```tsx
<div id="new" />
```

`id` 新旧值不同，所以执行：

```txt
setProp(dom, "id", "new")
```

---

## 14. setProp：真正设置 DOM 属性

源码：

```ts
export function setProp(dom: Instance, prop: string, value: any) {
  switch (prop) {
    case 'child': {
      if (typeof value === 'string' || typeof value === 'number') {
        dom.textContent = value.toString();
      }
      break;
    }
    default: {
      dom.setAttribute(prop, value);
    }
  }
}
```

`commitHostUpdate` 本身不直接调用：

```ts
dom.setAttribute(...)
```

而是通过 host config 的：

```ts
setProp(dom, key, value)
```

这样 reconciler 不直接绑定太多 DOM 细节。

在当前教学版里，默认情况就是：

```ts
dom.setAttribute(prop, value);
```

---

## 15. HostText：commitTextUpdate

文本节点更新更直接。

commit 阶段：

```ts
case HostText: {
  if (flags & Update) {
    const newText = finishedWork.pendingProps;
    commitTextUpdate(finishedWork.stateNode, newText);
  }
}
```

host config：

```ts
export function commitTextUpdate(textInstance: TextInstance, text: string) {
  textInstance.nodeValue = text;
}
```

也就是说：

```txt
HostText Fiber.pendingProps
  ↓
Text.nodeValue
```

例子：

旧：

```tsx
<div>hello</div>
```

新：

```tsx
<div>world</div>
```

文本 Fiber 打 `Update`。

commit 阶段执行：

```txt
textNode.nodeValue = "world"
```

---

## 16. Update 完整流程：props 更新

旧：

```tsx
<button id="save" disabled>
  Save
</button>
```

新：

```tsx
<button id="submit">
  Save
</button>
```

### diff 阶段

`button` 的 `key/type` 没变，可以复用旧 Fiber。

创建 workInProgress：

```txt
new button Fiber.alternate -> old button Fiber
new button Fiber.stateNode -> old button DOM
```

### complete 阶段

比较 props：

```txt
oldProps !== newProps
```

于是：

```txt
button Fiber.flags |= Update
```

### commit 阶段

执行：

```ts
commitHostUpdate(dom, oldProps, newProps);
```

删除旧属性：

```txt
disabled 旧有新无 -> 清掉
```

设置新属性：

```txt
id: save -> submit
```

---

## 17. Update 完整流程：文本更新

旧：

```tsx
<span>old</span>
```

新：

```tsx
<span>new</span>
```

### diff 阶段

`span` 可以复用。

文本 child 也可以复用为 `HostText Fiber`。

### complete 阶段

文本 Fiber 比较：

```txt
oldProps = "old"
newProps = "new"
```

不同，所以：

```txt
HostText Fiber.flags |= Update
```

### commit 阶段

执行：

```ts
commitTextUpdate(finishedWork.stateNode, "new");
```

真实 DOM 变成：

```txt
Text.nodeValue = "new"
```

---

## 18. Update 和 Placement 的关系

`Placement` 关心位置：

```txt
这个 DOM 要插到哪里
这个 DOM 要移动到哪里
```

`Update` 关心内容：

```txt
这个 DOM 的属性或文本要改成什么
```

它们可以同时存在。

比如一个新创建的节点：

```tsx
<div id="box">hello</div>
```

它可能有：

```txt
Placement   要插入页面
```

但初始 props 通常在 mount 创建 DOM 时已经通过 `setInitialProps` 设置了，不一定需要 `Update`。

如果某个复用节点既移动又改属性，就可能同时有：

```txt
Placement | Update
```

commit 阶段会按当前代码顺序处理：

```txt
先 Placement
再 Update
```

---

## 19. Update 和 ChildDeletion 的关系

`ChildDeletion` 表示：

```txt
当前 Fiber 有旧 child 要删除
```

`Update` 表示：

```txt
当前 Fiber 自己对应的 DOM 要更新
```

所以主语不一样：

```txt
Update          我自己要更新
ChildDeletion   我有孩子要删除
```

比如：

旧：

```tsx
<div id="old">
  <span>A</span>
</div>
```

新：

```tsx
<div id="new" />
```

这个 `div Fiber` 可能同时有：

```txt
Update          id 从 old 变 new
ChildDeletion   span child 要删除
```

---

## 20. 为什么 Update 在 completeWork 阶段

children diff 主要解决：

```txt
子节点结构怎么变
```

也就是：

```txt
哪个 child 复用
哪个 child 插入
哪个 child 移动
哪个 child 删除
```

而 `Update` 需要在当前 Fiber 完成时，根据：

```txt
旧 Fiber 的 props
新 Fiber 的 props
```

判断当前节点自身是否需要更新。

所以它放在 `completeWork` 里很自然。

可以这样分工：

```txt
beginWork / ChildFiber    处理 children 结构变化
completeWork             处理当前 Host 节点的创建或更新标记
commitWork               真正修改 DOM
```

---

## 21. 当前代码里的注意点

### oldProps 的来源

当前代码：

```ts
const oldProps = current?.pendingProps || null;
const newProps = workInProgress.pendingProps;
```

教学版可以先这样理解。

真实 React 中还会区分：

```txt
pendingProps    本次渲染传入的新 props
memoizedProps   上一次提交后的 props
```

真实比较通常会拿旧的 memoized props 和新的 pending props 做对比。

当前项目还没有完整引入 `memoizedProps`，所以先用 `current.pendingProps` 来理解即可。

### props 比较是引用比较

当前代码：

```ts
if (oldProps !== newProps) {
  markUpdate(workInProgress);
}
```

这会让很多情况都打 `Update`。

比如每次 JSX 生成的 props 对象都是新对象：

```txt
oldProps !== newProps
```

即使内部值没变，也可能打 Update。

不过 commit 阶段的 `commitHostUpdate` 会再逐个 prop 比较值。

### setProp 删除属性时还不完整

当前删除旧属性时：

```ts
setProp(dom, oldKey, null);
```

但 `setProp` 默认是：

```ts
dom.setAttribute(prop, value);
```

这对 `null` 的处理还比较简化。

更完整的 DOM 实现通常会区分：

```txt
removeAttribute
事件属性
style
className
boolean 属性
children/textContent
```

教学版先抓住主线：

```txt
commitHostUpdate 负责把 prop 差异应用到 DOM
```

### children 字段的小细节

当前 `setProp` 里判断的是：

```ts
case 'child':
```

但 JSX props 里通常是：

```txt
children
```

所以后续修属性更新时，可以把这里也一起校正。

---

## 22. 本讲小结

这一讲重点是：

- `Update` 表示当前 Fiber 对应的 DOM 需要更新
- `Update` 主要由 `completeWork` 产生
- `HostComponent` 比较 props，变化时打 `Update`
- `HostText` 比较文本内容，变化时打 `Update`
- `Update` 打在当前 Fiber 自己身上
- `bubbleProperties` 会把子树里的 `Update` 冒泡到父级 `subtreeFlags`
- commit 阶段通过 `commitHostUpdate` 更新 DOM props
- commit 阶段通过 `commitTextUpdate` 更新文本节点
- `Placement` 关心位置，`Update` 关心内容，`ChildDeletion` 关心删除子节点

一句话总结：

> Update 的本质，是 completeWork 阶段发现复用的宿主节点内容发生变化，于是在当前 Fiber 上打标记，等 commit 阶段再把 props 或文本差异应用到真实 DOM。



