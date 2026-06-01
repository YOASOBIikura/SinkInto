---
title: React-08
date: 2026-06-01 17:12:22
categories:
  - ReactCore
tags:
  - Placement 标记
---

核心文件：

- `packages/reconciler/ChildFiber.ts`
- `packages/reconciler/CommitWork.ts`
- `packages/reconciler/FiberFlags.ts`

这一讲要回答的问题是：

> React diff 发现节点需要插入或移动之后，是怎么打 Placement 标记，并在 commit 阶段操作 DOM 的？

前面我们已经知道：

```txt
diff 阶段      负责比较新旧 children，并打 flags
complete 阶段  负责冒泡 subtreeFlags
commit 阶段    负责根据 flags 操作真实 DOM
```

`Placement` 就是 diff 阶段产生、commit 阶段消费的一种 mutation 标记。

它表示：

```txt
这个 Fiber 对应的 DOM 需要被放到正确位置
```

这个“放到正确位置”包含两种情况：

```txt
插入新节点
移动旧节点
```

---

## 1. Placement 是什么

源码：

```ts
export type Flags = number;

export const NoFlags = 0b0000000;

export const Placement = 0b0000010;
export const Update = 0b0000100;
export const ChildDeletion = 0b0001000;

export const MutationMask = Placement | Update | ChildDeletion;
```

`Placement` 是一个 mutation flag。

它表示当前 Fiber 在 commit mutation 阶段需要执行位置相关操作。

可以先记成：

```txt
Placement = 插入或移动
```

---

## 2. Placement 打在哪里

`Placement` 和 `ChildDeletion` 不一样。

`ChildDeletion` 打在父 Fiber 上，因为被删除的旧 Fiber 不会出现在新树里。

`Placement` 打在新 Fiber 自己身上，因为需要插入或移动的 Fiber 会出现在新的 workInProgress 树里。

对比一下：

```txt
Placement
  打在 newFiber.flags 上
  commit 阶段遍历新树时能找到它

ChildDeletion
  打在 parentFiber.flags 上
  被删除节点放在 parentFiber.deletions 中
```

所以：

```txt
插入/移动：看当前 Fiber.flags
删除：看父 Fiber.deletions
```

---

## 3. Placement 的产生位置

当前代码里主要有三个地方会打 `Placement`。

### 单节点创建时

源码：

```ts
const created = createFiberFromElement(children);
created.return = returnFiber;
created.flags |= Placement;
return created;
```

如果单节点 diff 没有找到可复用旧 Fiber，就会创建新 Fiber。

新 Fiber 没有对应旧 DOM，所以需要插入。

因此打：

```ts
created.flags |= Placement;
```

### 单节点外层 placeSingleChild

源码：

```ts
function placeSingleChild(newFiber: Fiber) {
  if (newFiber.alternate === null) {
    newFiber.flags |= Placement;
  }

  return newFiber;
}
```

这里判断：

```txt
newFiber.alternate === null
```

如果没有 `alternate`，说明它不是从旧 Fiber 复用来的。

也就是：

```txt
这是一个新节点
```

所以需要打 `Placement`。

### 数组 diff 的 placeChild

数组 children 里的每个新 Fiber 都会经过：

```ts
lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
```

`placeChild` 是数组 diff 里判断插入和移动的核心。

---

## 4. placeSingleChild：单节点插入

源码：

```ts
function placeSingleChild(newFiber: Fiber) {
  if (newFiber.alternate === null) {
    newFiber.flags |= Placement;
  }

  return newFiber;
}
```

单节点场景比较简单。

如果能复用：

```txt
newFiber.alternate !== null
```

说明旧 DOM 可以继续用，不需要插入。

如果不能复用：

```txt
newFiber.alternate === null
```

说明是新创建的 Fiber，需要把它对应的 DOM 插入页面。

所以：

```txt
无 alternate -> Placement
有 alternate -> 不打 Placement
```

---

## 5. placeChild：数组中的插入和移动

源码：

```ts
function placeChild(newFiber: Fiber, lastPlaceIndex: number, newIndex: number): number {
  newFiber.index = newIndex;

  const current = newFiber.alternate;

  if (current !== null) {
    const oldIndex = current.index;

    if (oldIndex < lastPlaceIndex) {
      newFiber.flags |= Placement;
      return lastPlaceIndex;
    } else {
      return oldIndex;
    }
  } else {
    newFiber.flags |= Placement;
    return lastPlaceIndex;
  }
}
```

它做三件事：

```txt
1. 给 newFiber 设置新的 index
2. 判断它是新插入还是复用旧节点
3. 如果复用了旧节点，再判断是否需要移动
```

---

## 6. 新节点插入

判断条件：

```ts
const current = newFiber.alternate;

if (current === null) {
  newFiber.flags |= Placement;
}
```

如果 `current === null`，说明：

```txt
这个 newFiber 没有对应的旧 Fiber
```

也就是：

```txt
旧树里没有这个节点
```

所以它需要被插入到 DOM 中。

例子：

```txt
旧：[A, B]
新：[A, B, C]
```

`A`、`B` 可以复用。

`C` 是新节点：

```txt
C.alternate === null
C.flags |= Placement
```

---

## 7. 旧节点移动

如果：

```ts
current !== null
```

说明这个新 Fiber 是从旧 Fiber 复用来的。

这时不一定需要插入，但可能需要移动。

判断移动靠的是：

```ts
if (oldIndex < lastPlaceIndex) {
  newFiber.flags |= Placement;
  return lastPlaceIndex;
}
```

如果旧位置 `oldIndex` 小于 `lastPlaceIndex`，说明它在旧数组里靠前，但在新数组里被放到了某个稳定节点后面。

所以它需要移动。

---

## 8. lastPlacedIndex 怎么理解

`lastPlacedIndex` 可以理解成：

> 到目前为止，已经确认不需要移动的旧节点里，最靠后的旧位置。

也可以记成：

```txt
当前稳定序列里最大的 oldIndex
```

如果后面遇到的旧节点 `oldIndex` 比它还小，说明这个节点逆序了。

逆序就需要移动。

判断规则：

```txt
oldIndex < lastPlacedIndex   需要移动，打 Placement
oldIndex >= lastPlacedIndex  不需要移动，更新 lastPlacedIndex
```

---

## 9. 移动例子：A B C 到 A C B

旧：

```txt
[A, B, C]
```

新：

```txt
[A, C, B]
```

旧 index：

```txt
A: 0
B: 1
C: 2
```

### 处理 A

```txt
oldIndex = 0
lastPlacedIndex = 0
0 < 0 ? false
```

`A` 不移动。

更新：

```txt
lastPlacedIndex = 0
```

### 处理 C

```txt
oldIndex = 2
lastPlacedIndex = 0
2 < 0 ? false
```

`C` 不移动。

更新：

```txt
lastPlacedIndex = 2
```

### 处理 B

```txt
oldIndex = 1
lastPlacedIndex = 2
1 < 2 ? true
```

`B` 需要移动。

所以：

```txt
B.flags |= Placement
```

最终：

```txt
A 不动
C 不动
B 移动
```

这里的思路是：

```txt
保留尽可能多的稳定节点
移动逆序的节点
```

---

## 10. 移动例子：A B C 到 B A C

旧：

```txt
[A, B, C]
```

新：

```txt
[B, A, C]
```

旧 index：

```txt
A: 0
B: 1
C: 2
```

### 处理 B

```txt
oldIndex = 1
lastPlacedIndex = 0
1 < 0 ? false
```

`B` 不移动。

更新：

```txt
lastPlacedIndex = 1
```

### 处理 A

```txt
oldIndex = 0
lastPlacedIndex = 1
0 < 1 ? true
```

`A` 需要移动。

所以：

```txt
A.flags |= Placement
```

### 处理 C

```txt
oldIndex = 2
lastPlacedIndex = 1
2 < 1 ? false
```

`C` 不移动。

最终：

```txt
B 不动
A 移动
C 不动
```

注意：

```txt
不是谁在新数组里排到前面，谁就一定移动
而是谁的 oldIndex 出现了逆序，谁移动
```

---

## 11. Placement 和 Fiber.index

数组 diff 里每个新 Fiber 都会设置：

```ts
newFiber.index = newIndex;
```

这个 `index` 表示：

```txt
当前 Fiber 在新 children 数组中的位置
```

旧 Fiber 上也有 `index`。

所以更新时可以拿到：

```ts
const oldIndex = current.index;
```

其中：

```txt
current             旧 Fiber
current.index       旧位置
newFiber.index      新位置
```

`placeChild` 正是通过旧位置和 `lastPlacedIndex` 来判断是否移动。

---

## 12. Placement 和 alternate

`alternate` 是判断插入还是移动的关键。

```txt
newFiber.alternate === null
```

表示：

```txt
新插入
```

因为没有旧 Fiber 可以复用。

```txt
newFiber.alternate !== null
```

表示：

```txt
复用旧节点
```

这时如果打了 `Placement`，通常表示移动。

所以可以记成：

```txt
Placement + 无 alternate   插入
Placement + 有 alternate   移动
```

不过 commit 阶段不需要特别区分插入和移动。

因为 DOM 的 `appendChild` 和 `insertBefore` 对已经存在的 DOM 节点也能起到移动效果。

---

## 13. completeWork 如何让 Placement 被找到

diff 阶段只是给某些 Fiber 打了：

```ts
newFiber.flags |= Placement;
```

commit 阶段从根节点开始遍历。

为了快速知道子树里有没有副作用，`completeWork` 会冒泡 `subtreeFlags`。

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
Placement
```

它的父级 `subtreeFlags` 也会包含 `Placement`。

这样 commit 阶段看到：

```ts
finishedWork.subtreeFlags & MutationMask
```

就知道要继续进入子树处理。

---

## 14. commit 阶段如何消费 Placement

入口在 `CommitWork.ts`：

```ts
function commitMutaionEffectsOnFiber(finishedWork: Fiber) {
  const flags = finishedWork.flags;

  recursivelyTraversMutationEffects(finishedWork);

  if (flags & Placement) {
    commitHostPlacement(finishedWork);
    finishedWork.flags &= ~Placement;
  }

  // Update ...
}
```

处理步骤：

```txt
1. 判断当前 Fiber 是否有 Placement
2. 有的话执行 commitHostPlacement
3. 执行完清除 Placement 标记
```

清除标记：

```ts
finishedWork.flags &= ~Placement;
```

表示：

```txt
这个位置副作用已经被提交过了
```

---

## 15. commitHostPlacement 做什么

源码：

```ts
function commitHostPlacement(finishedWork: Fiber) {
  const parentFiber = getHostParentFiber(finishedWork);
  const before = getHostSibling(parentFiber!);

  switch (parentFiber!.tag) {
    case HostComponent: {
      const parent = parentFiber!.stateNode;
      insertOrAppendPlacementNode(finishedWork, parent, before);
      break;
    }
    case HostRoot: {
      const parent = parentFiber!.stateNode.containerInfo;
      insertOrAppendPlacementNodeIntoContainer(finishedWork, parent, before);
      break;
    }
  }
}
```

它主要做三件事：

```txt
1. 找最近的宿主父 Fiber
2. 找稳定的宿主兄弟节点 before
3. 把 finishedWork 对应的 DOM 插入到父 DOM 里
```

---

## 16. 找最近的宿主父节点

并不是每个 Fiber 都有真实 DOM。

比如：

```txt
FunctionComponent 没有真实 DOM
HostComponent 有真实 DOM
HostRoot 有根容器
HostText 有文本 DOM
```

所以 Placement 提交时，要先向上找最近的宿主父节点。

源码：

```ts
function isHostParent(parent: Fiber): boolean {
  return parent.tag === HostComponent || parent.tag === HostRoot;
}
```

```ts
function getHostParentFiber(finishedWork: Fiber): Fiber | null {
  let parent = finishedWork.return;

  while (parent) {
    if (isHostParent(parent)) {
      return parent;
    }

    parent = parent.return;
  }

  return null;
}
```

例子：

```tsx
function App() {
  return <span>hello</span>;
}
```

如果要插入的是 `span Fiber`，它向上可能经过：

```txt
span Fiber
  return -> App Fiber
  return -> HostRoot Fiber
```

`App Fiber` 没有 DOM，要继续向上找 `HostRoot`。

---

## 17. 找稳定的宿主兄弟节点

插入 DOM 时有两种方式：

```txt
appendChild       追加到末尾
insertBefore      插入到某个兄弟节点前面
```

如果新节点后面存在一个稳定的 DOM 节点，就可以：

```ts
insertBefore(parent, child, before);
```

如果找不到稳定兄弟，就：

```ts
appendChild(parent, child);
```

当前代码里找稳定兄弟的函数是：

```ts
function getHostSibling(fiber: Fiber): any
```

它要找的是：

```txt
当前 Placement 节点后面，第一个没有 Placement 标记的 HostComponent 或 HostText
```

为什么要跳过带 `Placement` 的兄弟？

因为带 `Placement` 的兄弟自己也还没放到最终位置。

它不是稳定参照物。

---

## 18. getHostSibling 的核心思路

源码摘出来看：

```ts
function getHostSibling(fiber: Fiber): any {
  let node: Fiber = fiber;

  sibling: while (node) {
    while (!node.sibling) {
      if (isHostParent(node)) {
        return null;
      }
      node = node.return!;
    }

    node = node.sibling;

    while (node.tag !== HostComponent && node.tag !== HostText) {
      if (node.flags & Placement) {
        continue sibling;
      }

      if (!node.child) {
        continue sibling;
      } else {
        node = node.child;
      }
    }

    if (!(node!.flags & Placement)) {
      return node!.stateNode;
    }
  }
}
```

它的查找逻辑是：

```txt
1. 从当前节点开始向后找 sibling
2. 如果没有 sibling，就向上回到父节点继续找
3. 找到非 Host 节点时，尝试进入它的 child
4. 如果节点带 Placement，跳过它
5. 找到第一个不带 Placement 的 HostComponent 或 HostText，返回它的 stateNode
6. 如果一直找不到，返回 null
```

返回值：

```txt
有稳定兄弟   返回 before DOM
没有稳定兄弟 返回 null
```

---

## 19. 插入 HostComponent 或 HostText

真正插入普通宿主节点的函数是：

```ts
function insertOrAppendPlacementNode(node: Fiber, parent: Instance, before: Instance | null) {
  const { tag } = node;
  const isHost = tag === HostComponent || tag === HostText;

  if (isHost) {
    const stateNode = node.stateNode;

    if (before) {
      insertBefore(parent, stateNode, before);
    } else {
      appendChild(parent, stateNode);
    }

    return;
  }

  const child = node.child;

  if (child) {
    insertOrAppendPlacementNode(child, parent, before);
    let sibling = child.sibling;
    while (sibling) {
      insertOrAppendPlacementNode(sibling, parent, before);
      sibling = sibling.sibling;
    }
  }
}
```

如果当前节点是：

```txt
HostComponent
HostText
```

就直接插入它的 `stateNode`。

如果当前节点是函数组件：

```txt
FunctionComponent
```

它自己没有 DOM。

所以要递归插入它下面的真实宿主节点。

---

## 20. 插入 FunctionComponent

例子：

```tsx
function Child() {
  return <span>hello</span>;
}

<div>
  <Child />
</div>
```

如果 `Child Fiber` 打了 `Placement`，它自己没有 DOM。

真正要插入的是它下面的：

```txt
span DOM
```

所以 `insertOrAppendPlacementNode` 会：

```txt
遇到 FunctionComponent
  进入 child
  找到 HostComponent span
  插入 span.stateNode
```

这就是为什么 Placement 不只处理 HostComponent，也要处理函数组件子树。

---

## 21. 插入到根容器

如果最近的宿主父节点是 `HostRoot`：

```ts
case HostRoot: {
  const parent = parentFiber!.stateNode.containerInfo;
  insertOrAppendPlacementNodeIntoContainer(finishedWork, parent, before);
  break;
}
```

容器可能是：

```txt
root div
document html
```

当前代码里对 `HTML` 做了特殊处理：

```ts
function appendChildtoContainer(parent: Instance, child: Instance) {
  if (parent.nodeName === 'HTML') {
    appendChild(parent.ownerDocument.body, child);
  } else {
    appendChild(parent, child);
  }
}
```

如果容器是 `HTML`，就实际插到：

```txt
document.body
```

里。

---

## 22. appendChild 和 insertBefore 都能移动节点

浏览器 DOM 有一个特点：

如果一个 DOM 节点已经存在于父节点下面，再对它执行：

```ts
parent.appendChild(child)
```

或：

```ts
parent.insertBefore(child, before)
```

浏览器会把这个已有节点移动到新位置。

所以 commit 阶段可以用同一套逻辑处理：

```txt
新节点插入
旧节点移动
```

对于新节点：

```txt
DOM 之前不在页面里 -> 插入
```

对于旧节点：

```txt
DOM 已经在页面里 -> 移动
```

这就是为什么 `Placement` 可以同时表示插入和移动。

---

## 23. Placement 完整流程

用一个插入例子串起来。

旧：

```txt
[A, B]
```

新：

```txt
[A, B, C]
```

### diff 阶段

`A`、`B` 复用。

`C` 是新节点：

```txt
C.alternate === null
C.flags |= Placement
```

### complete 阶段

`bubbleProperties` 冒泡：

```txt
父 Fiber.subtreeFlags 包含 Placement
```

### commit 阶段

遍历到 `C Fiber`：

```ts
if (flags & Placement) {
  commitHostPlacement(C);
}
```

找到父 DOM。

找不到稳定兄弟：

```txt
before = null
```

执行：

```txt
appendChild(parentDOM, cDOM)
```

最后清除：

```ts
C.flags &= ~Placement;
```

---

## 24. Placement 移动完整例子

旧：

```txt
[A, B, C]
```

新：

```txt
[A, C, B]
```

### diff 阶段

`A` 复用，不动。

`C` 复用，不动。

`B` 复用，但：

```txt
oldIndex = 1
lastPlacedIndex = 2
1 < 2
```

所以：

```txt
B.flags |= Placement
```

### commit 阶段

遍历到 `B Fiber`。

因为 `B` 是 HostComponent 或下面能找到 HostComponent，所以拿到它的 DOM。

找到稳定的 host sibling。

如果没有稳定 sibling，就 append。

如果有稳定 sibling，就 insertBefore。

浏览器会把原来的 `B DOM` 移动到新位置。

---

## 25. 和 Update 的区别

`Placement` 关心位置：

```txt
这个 DOM 放在哪里
```

`Update` 关心内容：

```txt
这个 DOM 的 props/text 是否变化
```

一个 Fiber 可以同时有：

```txt
Placement | Update
```

比如新建一个节点，同时它还有初始属性。

当前 commit 阶段会先处理 Placement，再根据 Fiber 类型处理 Update。

---

## 26. 和 ChildDeletion 的区别

`Placement`：

```txt
打在当前新 Fiber 上
当前新 Fiber 在新树里
commit 阶段处理当前 Fiber
```

`ChildDeletion`：

```txt
打在父 Fiber 上
被删除旧 Fiber 不在新树里
commit 阶段处理父 Fiber.deletions
```

所以可以这样记：

```txt
Placement 是“我自己要被放到正确位置”
ChildDeletion 是“我有旧孩子要被删掉”
```

---

## 27. 当前代码里的注意点

复习时先理解主流程，后面修代码时可以留意这些点。

### 单节点可能重复打 Placement

当前单节点新建时已经有：

```ts
created.flags |= Placement;
```

外层又会执行：

```ts
return placeSingleChild(reconcileSingleElement(fiber, children));
```

`placeSingleChild` 里也可能打一次：

```ts
newFiber.flags |= Placement;
```

由于 flag 是按位或：

```ts
flags |= Placement
```

重复打不会改变最终结果。

但从结构上看，通常保留一个统一打标记的位置会更清晰。

### commitHostPlacement 找 sibling 的起点

当前代码：

```ts
const parentFiber = getHostParentFiber(finishedWork);
const before = getHostSibling(parentFiber!);
```

更合理的思路是从当前要插入或移动的节点开始找稳定兄弟：

```ts
const before = getHostSibling(finishedWork);
```

因为要找的是：

```txt
finishedWork 后面的稳定宿主兄弟
```

而不是父节点后面的兄弟。

### 容器插入函数递归时要保持容器逻辑

当前 `insertOrAppendPlacementNodeIntoContainer` 在递归子节点时调用的是：

```ts
insertOrAppendPlacementNode(child, parent, before);
```

如果要保持根容器的特殊处理逻辑，可以考虑递归调用容器版本：

```ts
insertOrAppendPlacementNodeIntoContainer(child, parent, before);
```

这类细节不影响先理解 `Placement` 的核心概念，但后面调 DOM 插入位置时很关键。

---

## 28. 本讲小结

这一讲重点是：

- `Placement` 表示插入或移动
- 新 Fiber 没有 `alternate`，说明是插入
- 复用 Fiber 但 `oldIndex < lastPlacedIndex`，说明要移动
- 数组 diff 通过 `placeChild` 判断是否打 `Placement`
- `lastPlacedIndex` 表示当前稳定序列里最大的旧位置
- `completeWork` 会通过 `subtreeFlags` 冒泡 `Placement`
- commit 阶段通过 `commitHostPlacement` 消费 `Placement`
- 插入前要找最近宿主父节点和稳定宿主兄弟节点
- FunctionComponent 自己没有 DOM，Placement 时要插入它下面的宿主节点
- DOM 的 `appendChild` / `insertBefore` 可以同时处理插入和移动

一句话总结：

> Placement 的本质，是 diff 阶段发现某个新 Fiber 对应的 DOM 需要被放到正确位置，并把这个位置操作延迟到 commit 阶段统一执行。



