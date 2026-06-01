---
title: React-08
date: 2026-06-01 17:06:10
categories:
  - ReactCore
tags:
  - Delete 和 ChildDeletion
---

# 第 10 讲：Delete标记

核心文件：

- `packages/reconciler/ChildFiber.ts`
- `packages/reconciler/CompleteWork.ts`
- `packages/reconciler/CommitWork.ts`
- `packages/reconciler/FiberFlags.ts`

这一讲要回答的问题是：

> React diff 发现节点要删除之后，是怎么记录删除、提交删除、清理引用的？

前面我们已经知道：

```txt
diff 阶段      负责比较新旧 children，生成新的 Fiber 子链表
complete 阶段  负责完成 Fiber，并冒泡 flags
commit 阶段    负责根据 flags 操作真实 DOM
```

删除逻辑横跨这三个阶段：

```txt
ChildFiber    标记 ChildDeletion，记录 deletions
CompleteWork  冒泡 subtreeFlags
CommitWork    删除真实 DOM，断开 Fiber 引用
```

---

## 1. 先记住一句话

删除和插入不一样。

插入时，新 Fiber 会出现在新的 workInProgress 树里，所以可以把 `Placement` 打在新 Fiber 自己身上。

删除时，被删除的旧 Fiber 不会出现在新的 workInProgress 树里。

所以不能把删除标记只打在被删除节点自己身上。

当前实现采用的是：

```txt
ChildDeletion 打在父 Fiber 上
被删除的旧 Fiber 放进父 Fiber.deletions 数组
```

结构大概是：

```txt
parent Fiber
  flags: ChildDeletion
  deletions: [old child A, old child B]
```

这就是删除逻辑里最重要的设计点。

---

## 2. 删除标记是什么

源码：

```ts
export type Flags = number;

export const NoFlags = 0b0000000;

export const Placement = 0b0000010;
export const Update = 0b0000100;
export const ChildDeletion = 0b0001000;

export const MutationMask = Placement | Update | ChildDeletion;
```

`ChildDeletion` 是一个 mutation flag。

它表示：

```txt
当前 Fiber 有子节点需要删除
```

注意这句话里的主语是：

```txt
当前 Fiber
```

不是：

```txt
当前 Fiber 自己要删除
```

也就是说：

```txt
fiber.flags & ChildDeletion
```

意思是：

```txt
fiber.deletions 里有需要删除的 child
```

---

## 3. 删除在哪里产生

删除在 diff 阶段产生。

核心函数在 `ChildFiber.ts`：

```ts
function deleteChild(returnFiber: Fiber, childToDelete: Fiber) {
  const deletions = returnFiber.deletions;

  if (deletions === null) {
    returnFiber.deletions = [childToDelete];
    returnFiber.flags |= ChildDeletion;
  } else {
    deletions.push(childToDelete);
  }
}
```

参数含义：

```txt
returnFiber     新树里的父 Fiber
childToDelete   旧树里要删除的 child Fiber
```

它做两件事：

```txt
1. 把 childToDelete 放到 returnFiber.deletions
2. 给 returnFiber 打 ChildDeletion 标记
```

第一次删除时：

```ts
returnFiber.deletions = [childToDelete];
returnFiber.flags |= ChildDeletion;
```

后续还有删除节点时：

```ts
deletions.push(childToDelete);
```

---

## 4. 为什么删除标记打在父 Fiber 上

假设旧 children 是：

```tsx
<div>
  <p key="a">A</p>
  <p key="b">B</p>
</div>
```

新 children 是：

```tsx
<div>
  <p key="a">A</p>
</div>
```

更新后的 workInProgress 子链表只会有：

```txt
new div Fiber
  child -> new p a Fiber
```

旧的 `p b Fiber` 不会挂在新树里。

如果把删除标记打在 `old p b Fiber` 自己身上，commit 阶段遍历新树时就很难找到它。

所以要把它记录到父 Fiber 上：

```txt
new div Fiber
  flags: ChildDeletion
  deletions: [old p b Fiber]
  child -> new p a Fiber
```

这样 commit 阶段遍历到 `new div Fiber` 时，就能处理它的 `deletions`。

---

## 5. deleteRemainingChildren：删除一串旧兄弟

源码：

```ts
function deleteRemainingChildren(returnFiber: Fiber, childrenToDelete: Fiber | null) {
  let childToDelete = childrenToDelete;

  while (childToDelete !== null) {
    deleteChild(returnFiber, childToDelete);
    childToDelete = childToDelete.sibling;
  }
}
```

它表示：

```txt
从 childrenToDelete 开始，沿着 sibling 链表全部删除
```

常见场景：

```txt
旧：[A, B, C]
新：[A]
```

`A` 复用后，旧的 `B`、`C` 都应该删除。

于是：

```ts
deleteRemainingChildren(returnFiber, child.sibling);
```

最终：

```txt
returnFiber.deletions = [old B, old C]
returnFiber.flags |= ChildDeletion
```

---

## 6. 单节点 diff 里的删除

单节点 diff 在：

```ts
function reconcileSingleElement(returnFiber: any, children: any): Fiber
```

### key 不同

源码：

```ts
if (child.key === key) {
  // ...
} else {
  deleteChild(returnFiber, child);
}
```

表示：

```txt
当前旧 child 的 key 和新节点 key 不同
当前旧 child 不可能复用
先把它标记删除
再继续看下一个旧 sibling
```

例子：

```txt
旧：[A, B]
新：B
```

先比较：

```txt
old A vs new B
```

`key` 不同，删除 `A`。

然后继续比较：

```txt
old B vs new B
```

如果 `type` 也相同，就复用 `B`。

### key 相同但 type 不同

源码：

```ts
if (elementType === child.type) {
  // 复用
} else {
  deleteRemainingChildren(returnFiber, child);
  break;
}
```

表示：

```txt
key 相同，说明已经找到对应位置
但 type 不同，当前旧 child 不能复用
并且后面的旧兄弟也都不需要了
```

所以从当前 child 开始全部删除。

### 找到可复用节点后删除剩余兄弟

源码：

```ts
const existing = createWorkInProgress(child, children.props);
existing.return = returnFiber;
existing.index = 0;
deleteRemainingChildren(returnFiber, child.sibling);
return existing;
```

新的 children 是单节点。

所以找到可复用节点后，它后面的旧 siblings 都要删掉。

---

## 7. 数组 diff 里的删除

数组 diff 里也有三类删除。

### 第一类：同位置 key 相同但 type 不同

源码：

```ts
if (oldFiber && newFiber.alternate === null) {
  deleteChild(returnFiber, oldFiber);
}
```

这里的意思是：

```txt
updateSlot 没有返回 null，说明 key 对上了
但 newFiber.alternate === null，说明 type 不同，创建了新 Fiber
所以当前位置的 oldFiber 要删除
```

例子：

```txt
旧：[<div key="a" />]
新：[<span key="a" />]
```

`key` 一样，所以还在同一个 slot 上。

但 `type` 不同，旧 `div` 不能复用，需要删除。

### 第二类：新数组已经结束

源码：

```ts
if (newIdx === children.length) {
  deleteRemainingChildren(returnFiber, oldFiber);
  return resultingFirstChild;
}
```

例子：

```txt
旧：[A, B, C]
新：[A, B]
```

`A`、`B` 处理完后，新数组结束，旧链表还剩 `C`。

所以删除剩余旧节点。

### 第三类：Map 最后剩余的旧节点

源码：

```ts
existingChildren.forEach((child: Fiber) => deleteChild(returnFiber, child));
```

进入 Map 阶段后：

```txt
能被新 children 找到的旧 Fiber，会从 Map 中删除
最后还留在 Map 里的旧 Fiber，就是新 children 不再需要的节点
```

所以它们都要删除。

---

## 8. 删除标记如何冒泡

`deleteChild` 只给父 Fiber 自己打了：

```ts
returnFiber.flags |= ChildDeletion;
```

但是 commit 阶段需要快速知道：

```txt
某个子树里有没有 mutation 副作用
```

所以 `completeWork` 里会冒泡 `subtreeFlags`。

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

如果某个子 Fiber 有 `ChildDeletion`：

```txt
child.flags 包含 ChildDeletion
```

父 Fiber 的：

```txt
subtreeFlags
```

也会包含它。

这样 commit 阶段可以通过：

```ts
finishedWork.subtreeFlags & MutationMask
```

判断是否需要继续往子树里找副作用。

---

## 9. Commit 阶段从哪里开始处理删除

入口：

```ts
export function commitMutaionEffects(finishedWork: Fiber) {
  commitMutaionEffectsOnFiber(finishedWork);
}
```

每个 Fiber 的 mutation 处理：

```ts
function commitMutaionEffectsOnFiber(finishedWork: Fiber) {
  const flags = finishedWork.flags;

  recursivelyTraversMutationEffects(finishedWork);

  if (flags & Placement) {
    commitHostPlacement(finishedWork);
    finishedWork.flags &= ~Placement;
  }

  // HostComponent / HostText Update ...
}
```

注意顺序：

```txt
先 recursivelyTraversMutationEffects
再处理当前 Fiber 自己的 Placement / Update
```

删除逻辑就在：

```ts
recursivelyTraversMutationEffects(finishedWork)
```

里面。

---

## 10. recursivelyTraversMutationEffects：先处理 deletions

源码：

```ts
function recursivelyTraversMutationEffects(finishedWork: Fiber) {
  const deletions = finishedWork.deletions;

  if (deletions !== null) {
    for (let i = 0; i < deletions.length; i++) {
      commitDeletionEffects(finishedWork, deletions[i] as Fiber);
    }
  }

  if (finishedWork.subtreeFlags & MutationMask) {
    let child = finishedWork.child;
    while (child) {
      commitMutaionEffectsOnFiber(child);
      child = child.sibling;
    }
  }
}
```

这个函数做两件事：

```txt
1. 如果当前 Fiber 有 deletions，先处理删除
2. 如果子树里还有 mutation 副作用，继续递归子 Fiber
```

也就是说，删除真正被消费的位置是：

```txt
finishedWork.deletions
```

不是被删除节点自己的 `flags`。

---

## 11. commitDeletionEffects：删除前先找宿主父节点

源码：

```ts
function commitDeletionEffects(finishedWork: Fiber, deletedFiber: Fiber) {
  let parent: null | Fiber = finishedWork;

  findParent: while (parent) {
    switch (parent.tag) {
      case HostComponent: {
        hostParent = parent.stateNode;
        hostParentIsContainer = false;
        break findParent;
      }
      case HostRoot: {
        hostParent = parent.stateNode.containerInfo;
        hostParentIsContainer = true;
        break findParent;
      }
    }

    parent = parent.return;
  }

  commitDeletionEffectsOnFiber(deletedFiber);

  hostParent = null;
  hostParentIsContainer = false;

  detachFiberMutation(deletedFiber);
}
```

删除 DOM 前，必须先知道：

```txt
要从哪个 DOM 父节点里删除
```

Fiber 父节点不一定是真实 DOM。

比如：

```tsx
function App() {
  return <span>hello</span>;
}
```

`App Fiber` 是函数组件，没有 DOM。

所以需要一路向上找最近的宿主父节点：

```txt
HostComponent   普通 DOM 节点，比如 div
HostRoot        根容器
```

找到后存在全局变量里：

```ts
hostParent = parent.stateNode;
hostParentIsContainer = false;
```

或：

```ts
hostParent = parent.stateNode.containerInfo;
hostParentIsContainer = true;
```

---

## 12. commitDeletionEffectsOnFiber：按节点类型删除

源码：

```ts
function commitDeletionEffectsOnFiber(deletedFiber: Fiber) {
  switch (deletedFiber.tag) {
    case HostRoot:
      recursivelyTraversDeletionEffects(deletedFiber);
      return;

    case FunctionComponent:
      recursivelyTraversDeletionEffects(deletedFiber);
      return;

    case HostComponent:
      const prevHostParent = hostParent;
      hostParent = null;
      recursivelyTraversDeletionEffects(deletedFiber);
      hostParent = prevHostParent;

      if (hostParent) {
        if (hostParentIsContainer) {
          commitHostRemoveChildFromContainer(deletedFiber);
        } else {
          commitHostRemoveChild(deletedFiber);
        }
      }
      return;

    case HostText:
      if (hostParentIsContainer) {
        commitHostRemoveChildFromContainer(deletedFiber);
      } else {
        commitHostRemoveChild(deletedFiber);
      }
      return;
  }
}
```

这里按 Fiber 类型分别处理。

---

## 13. 删除 FunctionComponent

函数组件没有真实 DOM。

所以删除函数组件时，不能直接：

```ts
removeChild(parent, deletedFiber.stateNode)
```

因为：

```txt
FunctionComponent.stateNode 通常是 null
```

它要做的是继续向下找真实 DOM：

```ts
case FunctionComponent:
  recursivelyTraversDeletionEffects(deletedFiber);
  return;
```

例子：

```tsx
function Child() {
  return <span>hello</span>;
}
```

删除 `Child Fiber` 时，真正要删的是它下面的：

```txt
span DOM
```

---

## 14. 删除 HostComponent

`HostComponent` 对应真实 DOM，比如：

```tsx
<div />
<span />
<p />
```

删除 HostComponent 时：

```ts
case HostComponent:
  const prevHostParent = hostParent;
  hostParent = null;
  recursivelyTraversDeletionEffects(deletedFiber);
  hostParent = prevHostParent;

  if (hostParent) {
    if (hostParentIsContainer) {
      commitHostRemoveChildFromContainer(deletedFiber);
    } else {
      commitHostRemoveChild(deletedFiber);
    }
  }
  return;
```

这里有一个容易迷糊的地方：

```ts
hostParent = null;
recursivelyTraversDeletionEffects(deletedFiber);
hostParent = prevHostParent;
```

它的意思是：

```txt
如果要删除的是一个 HostComponent
直接删除这个 HostComponent 对应的 DOM 就够了
它内部的 DOM 会随着父 DOM 一起被移除
```

比如删除：

```tsx
<div>
  <span>hello</span>
</div>
```

真正执行一次：

```txt
removeChild(parentDOM, divDOM)
```

就可以了。

不需要再单独：

```txt
removeChild(divDOM, spanDOM)
```

所以递归遍历子树时，先把 `hostParent` 置空，避免重复删除内部 DOM。

---

## 15. 删除 HostText

文本节点本身就是 DOM 节点。

源码：

```ts
case HostText:
  if (hostParentIsContainer) {
    commitHostRemoveChildFromContainer(deletedFiber);
  } else {
    commitHostRemoveChild(deletedFiber);
  }
  return;
```

如果父节点是普通 DOM：

```ts
removeChild(hostParent!, deletedFiber.stateNode);
```

如果父节点是根容器：

```ts
removeChild(parent!, deletedFiber.stateNode);
```

---

## 16. removeChild 是真正的 DOM 删除

普通宿主父节点：

```ts
function commitHostRemoveChild(deletedFiber: Fiber) {
  removeChild(hostParent!, deletedFiber.stateNode);
}
```

根容器：

```ts
function commitHostRemoveChildFromContainer(deletedFiber: Fiber) {
  let parent = null;

  if (hostParent?.nodeName === 'HTML') {
    parent = hostParent.ownerDocument.body;
  } else {
    parent = hostParent;
  }

  removeChild(parent!, deletedFiber.stateNode);
}
```

真正调用的是 host config：

```ts
export function removeChild(parent: Instance, child: Instance) {
  parent.removeChild(child);
}
```

到这里，DOM 已经从页面上删除。

---

## 17. detachFiberMutation：先断开 return

删除 DOM 后，会执行：

```ts
detachFiberMutation(deletedFiber);
```

源码：

```ts
function detachFiberMutation(deletedFiber: Fiber) {
  deletedFiber.return = null;

  if (deletedFiber.alternate !== null) {
    deletedFiber.alternate.return = null;
  }
}
```

这里先断开：

```txt
deletedFiber.return
deletedFiber.alternate.return
```

目的是让这个删除子树不再能向上连接到主 Fiber 树。

可以理解为：

```txt
先切断它和父节点的关系
```

---

## 18. commitPassiveUnmountEffects：删除后的引用清理

在 `WorkLoop.ts` 里，commit mutation 后会执行：

```ts
commitMutaionEffects(fiberRoot.current!.alternate!);
fiberRoot.current = fiberRoot.current!.alternate;
commitPassiveUnmountEffects(fiberRoot.current!);
```

删除 DOM 是 mutation 阶段做的。

删除后的引用清理在：

```ts
commitPassiveUnmountEffects(finishedWork)
```

里面做。

这一步可以理解成：

```txt
DOM 已经删了
接下来把 Fiber 之间不用的引用也断干净
```

---

## 19. recursivelyTraversPassiveUnmountEffects

源码：

```ts
function recursivelyTraversPassiveUnmountEffects(finishedWork: Fiber) {
  if (finishedWork.flags & ChildDeletion) {
    const deletions = finishedWork.deletions;

    if (deletions) {
      for (let i = 0; i < deletions.length; i++) {
        const childDelete = deletions[i];
        nextEffect = childDelete!;
        commitPassiveUnmountEffectsInsideOfDeletedTree_begin(childDelete!);
      }
    }

    detachAlternateSilbling(finishedWork);
    finishedWork.deletions = null;
  }

  if (finishedWork.subtreeFlags & ChildDeletion) {
    let child = finishedWork.child;
    while (child) {
      commitPassiveUnmountEffectsOnFiber(child);
      child = child.sibling;
    }
  }
}
```

它做两件事：

```txt
1. 如果当前 Fiber 有 ChildDeletion，就清理 deletions 里的删除子树
2. 如果子树里还有 ChildDeletion，就继续往下找
```

---

## 20. 清理删除子树

删除子树的清理分为 begin 和 complete。

开始：

```ts
function commitPassiveUnmountEffectsInsideOfDeletedTree_begin(childToDelete: Fiber) {
  while (nextEffect) {
    if (nextEffect.child) {
      nextEffect = nextEffect.child;
    } else {
      commitPassiveUnmountEffectsInsideOfDeletedTree_complete(childToDelete);
    }
  }
}
```

完成：

```ts
function commitPassiveUnmountEffectsInsideOfDeletedTree_complete(childToDelete: Fiber) {
  while (nextEffect) {
    const fiber = nextEffect;
    const sibling = fiber.sibling;
    const returnFiber = fiber.return;

    detachFiberAfterEffect(fiber);

    if (fiber === childToDelete) {
      nextEffect = null;
      return;
    }

    if (sibling) {
      nextEffect = sibling;
      return;
    }

    nextEffect = returnFiber;
  }
}
```

这是一种深度优先遍历。

大致顺序是：

```txt
先一路向下找到最深 child
清理当前 Fiber
有 sibling 就去 sibling
没有 sibling 就回到 return
直到回到 childToDelete
```

---

## 21. detachFiberAfterEffect：断开所有引用

源码：

```ts
function detachFiberAfterEffect(fiber: Fiber) {
  const alternate = fiber.alternate;

  if (alternate) {
    fiber.alternate = null;
    detachAlternateSilbling(alternate);
  }

  if (fiber.tag === HostComponent) {
    const dom = fiber.stateNode;
    if (dom) {
      detachDeletedInstance(dom);
    }
  }

  fiber.child = null;
  fiber.sibling = null;
  fiber.deletions = null;
  fiber.stateNode = null;
  fiber.return = null;
}
```

它会清理：

```txt
alternate
child
sibling
deletions
stateNode
return
```

如果是 HostComponent，还会清理 DOM 到 Fiber 的缓存关系：

```ts
detachDeletedInstance(dom);
```

这一步的目标是：

```txt
让删除掉的 Fiber 子树不再被主树或 DOM 引用
方便后续被垃圾回收
```

---

## 22. 完整删除流程

用一个例子串起来。

旧：

```tsx
<div>
  <span key="a">A</span>
  <span key="b">B</span>
</div>
```

新：

```tsx
<div>
  <span key="a">A</span>
</div>
```

### diff 阶段

数组 diff 发现新 children 已经结束，但旧链表还剩 `b`：

```ts
deleteRemainingChildren(returnFiber, oldFiber);
```

执行：

```txt
new div Fiber.flags |= ChildDeletion
new div Fiber.deletions = [old span b Fiber]
```

### complete 阶段

`bubbleProperties` 把子树里的 flags 往上冒泡。

父级可以通过：

```txt
subtreeFlags
```

知道下面有 mutation 副作用。

### commit mutation 阶段

遍历到 `new div Fiber`：

```ts
const deletions = finishedWork.deletions;
```

发现：

```txt
deletions = [old span b Fiber]
```

然后执行：

```ts
commitDeletionEffects(new div Fiber, old span b Fiber);
```

找到最近宿主父节点：

```txt
div DOM
```

真正删除：

```txt
removeChild(div DOM, span b DOM)
```

### passive unmount 阶段

DOM 删除后，再清理 Fiber 引用：

```txt
old span b Fiber.return = null
old span b Fiber.child = null
old span b Fiber.sibling = null
old span b Fiber.stateNode = null
old span b Fiber.alternate = null
```

---

## 23. 和 Placement 的区别

`Placement`：

```txt
打在新 Fiber 自己身上
因为新 Fiber 会出现在 workInProgress 树中
```

`ChildDeletion`：

```txt
打在父 Fiber 身上
因为被删除的旧 Fiber 不会出现在 workInProgress 树中
```

所以：

```txt
插入/移动：看当前 Fiber.flags
删除：看当前 Fiber.deletions
```

这就是两者最核心的区别。

---

## 24. 常见易混点

### 删除节点自己不一定有 ChildDeletion

比如：

```txt
old B 被删除
```

通常不是：

```txt
old B.flags |= ChildDeletion
```

而是：

```txt
parent.flags |= ChildDeletion
parent.deletions.push(old B)
```

### ChildDeletion 表示删除子节点

`ChildDeletion` 这个名字很准确。

它不是：

```txt
Deletion
```

而是：

```txt
ChildDeletion
```

也就是：

```txt
当前 Fiber 有 child 要删除
```

### 删除 DOM 和清理 Fiber 是两步

mutation 阶段：

```txt
removeChild 删除真实 DOM
```

passive unmount 阶段：

```txt
断开 Fiber 引用
清理 DOM 到 Fiber 的缓存
```

先让页面正确，再做内存引用清理。

---

## 25. 当前代码里的注意点

### `ChildDeletion` 要依赖 `deletions`

commit 阶段真正删除的是：

```ts
finishedWork.deletions
```

所以只打：

```ts
returnFiber.flags |= ChildDeletion;
```

还不够。

必须同时维护：

```ts
returnFiber.deletions
```

当前 `deleteChild` 已经做了这件事。

### 删除后要清理 `deletions`

当前 passive unmount 阶段有：

```ts
finishedWork.deletions = null;
```

这很重要。

否则父 Fiber 会一直保留旧删除子树的引用。

### `subtreeFlags` 决定是否继续遍历

commit 阶段是否进入子树，依赖：

```ts
finishedWork.subtreeFlags & MutationMask
```

所以 `completeWork` 的 `bubbleProperties` 对删除也很关键。

如果 `ChildDeletion` 没有被冒泡，上层就可能跳过这棵子树里的删除副作用。

---

## 26. 本讲小结

这一讲重点是：

- 删除由 diff 阶段产生
- `deleteChild` 会设置父 Fiber 的 `deletions`
- `ChildDeletion` 打在父 Fiber 上，不是打在被删除节点上
- `deleteRemainingChildren` 用来批量删除旧 sibling
- `completeWork` 通过 `subtreeFlags` 冒泡删除副作用
- commit mutation 阶段通过 `deletions` 删除真实 DOM
- 删除 FunctionComponent 时要继续向下找 Host 节点
- 删除 HostComponent 时删除它自己的 DOM 即可
- passive unmount 阶段负责断开 Fiber 引用

一句话总结：

> ChildDeletion 的本质，是让仍然留在新 Fiber 树里的父节点，记住那些已经不在新树里的旧子节点，并在 commit 阶段把它们从 DOM 和 Fiber 引用中移除。


