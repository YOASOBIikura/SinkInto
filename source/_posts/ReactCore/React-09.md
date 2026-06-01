---
title: React-08
date: 2026-06-01 13:34:42
categories:
  - ReactCore
tags:
  - Diff 算法
---

# 第 9 讲：Diff 算法

核心文件：

- `packages/reconciler/ChildFiber.ts`
- `packages/reconciler/Fiber.ts`
- `packages/reconciler/FiberFlags.ts`

这一讲要回答的问题是：

> 更新时，新 children 和旧 Fiber 子链表是怎么对比的？

前面我们已经知道：

```txt
current 树            页面上已经生效的 Fiber 树
workInProgress 树     正在计算中的下一棵 Fiber 树
alternate             同一个节点在两棵树之间的连接
```

到了更新阶段，React 不再只是“把 children 创建成 Fiber”，而是要尽量复用旧 Fiber。

这个过程就是 diff。

在当前教学版里，diff 的核心目标是：

> 根据新的 children 和旧的 Fiber 子链表，构建新的 workInProgress 子链表，并标记插入、移动、删除。

---

## 1. Diff 发生在哪里

diff 的入口仍然是：

```ts
reconcileChildFibers(fiber, children)
```

源码：

```ts
export function reconcileChildFibers(fiber: Fiber, children: any): Fiber | null {
  if (children.$$typeof === REACT_ELEMENT_TYPE) {
    return placeSingleChild(reconcileSingleElement(fiber, children));
  }

  if (Array.isArray(children)) {
    return reconcileChildrenArray(fiber, children);
  }

  return null;
}
```

它会根据 children 的类型分成两类：

```txt
单一 ReactElement   -> reconcileSingleElement
数组 children       -> reconcileChildrenArray
```

所以 diff 可以分开复习：

```txt
单节点 diff
数组 diff
```

---

## 2. Diff 对比的是谁

更新时，当前正在构建的是 workInProgress Fiber。

它的旧节点在：

```ts
returnFiber.alternate
```

旧子节点链表在：

```ts
returnFiber.alternate?.child
```

也就是说：

```txt
returnFiber                    新父 Fiber
returnFiber.alternate           旧父 Fiber
returnFiber.alternate.child     旧父 Fiber 的第一个旧子节点
```

比如旧结构是：

```tsx
<ul>
  <li key="a">A</li>
  <li key="b">B</li>
</ul>
```

旧 Fiber 链表大概是：

```txt
old ul Fiber
  child -> old li a
              sibling -> old li b
```

更新时，新 `ul Fiber` 会通过自己的 `alternate` 找到旧 `ul Fiber`：

```txt
new ul Fiber.alternate -> old ul Fiber
```

再从旧 `ul Fiber.child` 开始对比。

---

## 3. Diff 的复用条件

一个旧 Fiber 能不能复用，主要看两个东西：

```txt
key
type
```

### `key`

`key` 用来判断两个节点是不是同一个业务节点。

比如：

```tsx
<li key="a">A</li>
<li key="b">B</li>
```

如果下一次变成：

```tsx
<li key="b">B</li>
<li key="a">A</li>
```

React 可以通过 `key` 知道：

```txt
a 还是原来的 a
b 还是原来的 b
```

只是位置变了。

### `type`

`type` 用来判断节点类型是否一样。

比如：

```tsx
<div key="a" />
```

更新为：

```tsx
<span key="a" />
```

虽然 `key` 一样，但 `type` 从 `div` 变成了 `span`。

这时旧 `div Fiber` 不能复用，需要删除旧节点，再创建新节点。

### 复用规则

可以记成：

```txt
key 不同       不是同一个节点，不能复用
key 相同 type 不同   位置对应上了，但节点类型不同，不能复用
key 相同 type 相同   可以复用旧 Fiber
```

---

## 4. 复用旧 Fiber：createWorkInProgress

当旧 Fiber 可以复用时，会调用：

```ts
createWorkInProgress(oldFiber, pendingProps)
```

源码位置：

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
    workInProgress.flags = NoFlags;
    workInProgress.subtreeFlags = NoFlags;
    workInProgress.deletions = null;
  }

  workInProgress.memoizedState = current.memoizedState;
  return workInProgress;
}
```

这里的重点是：

```txt
复用不是直接修改旧 Fiber
而是基于旧 Fiber 创建或复用它的 alternate
```

复用后：

```txt
旧 Fiber.stateNode      真实 DOM
新 Fiber.stateNode      复用同一个真实 DOM
```

也就是说，Fiber 可以重新构建，但 DOM 可以尽量保留。

---

## 5. 单节点 Diff

单节点 diff 的函数是：

```ts
function reconcileSingleElement(returnFiber: any, children: any): Fiber
```

核心流程：

```ts
const key = children.key;
let child = returnFiber.alternate?.child;

while (child) {
  if (child.key === key) {
    const elementType = children.type;

    if (elementType === child.type) {
      const existing = createWorkInProgress(child, children.props);
      existing.return = returnFiber;
      existing.index = 0;
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
```

这个流程可以拆成几种情况。

---

## 6. 单节点情况一：key 和 type 都相同

旧：

```tsx
<div>
  <span key="a">old</span>
</div>
```

新：

```tsx
<div>
  <span key="a">new</span>
</div>
```

对比结果：

```txt
key:  a === a
type: span === span
```

可以复用旧 `span Fiber`。

代码：

```ts
const existing = createWorkInProgress(child, children.props);
existing.return = returnFiber;
existing.index = 0;
deleteRemainingChildren(returnFiber, child.sibling);
return existing;
```

这里还会删除旧节点后面的兄弟节点：

```ts
deleteRemainingChildren(returnFiber, child.sibling);
```

因为新的 children 是单节点。

如果旧 children 有多个，除了匹配上的这个节点，其余旧兄弟都应该删除。

---

## 7. 单节点情况二：key 相同，type 不同

旧：

```tsx
<div>
  <span key="a">old</span>
</div>
```

新：

```tsx
<div>
  <p key="a">new</p>
</div>
```

对比结果：

```txt
key:  a === a
type: span !== p
```

`key` 一样，说明找到了对应位置的旧节点。

但是 `type` 不同，旧节点不能复用。

代码：

```ts
deleteRemainingChildren(returnFiber, child);
break;
```

这表示：

```txt
从当前旧 child 开始，后面的旧节点全部删除
然后创建新的 p Fiber
```

---

## 8. 单节点情况三：key 不同

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
  <span key="b">B</span>
</div>
```

对比第一个旧节点：

```txt
old key: a
new key: b
```

`key` 不同，说明旧 `a` 不是新的这个节点。

代码：

```ts
deleteChild(returnFiber, child);
child = child.sibling;
```

继续向后找，直到找到 `key === b` 的旧 Fiber。

如果后面找到了，并且 `type` 也相同，就复用。

---

## 9. 数组 Diff 总览

数组 diff 的函数是：

```ts
function reconcileChildrenArray(returnFiber: any, children: any): Fiber | null
```

它分成三个阶段：

```txt
第一阶段：顺序比较
第二阶段：快速路径
第三阶段：Map 查找
```

为什么要分阶段？

因为大多数更新其实很简单：

```txt
旧：[A, B, C]
新：[A, B, C, D]
```

或者：

```txt
旧：[A, B, C]
新：[A, B]
```

这些不需要一开始就建 Map。

所以先走便宜的顺序比较，顺序断了以后，再进入 Map 查找。

---

## 10. 数组 Diff 的关键变量

源码：

```ts
let resultingFirstChild: Fiber | null = null;
let previousNewFiber: Fiber | null = null;

let oldFiber: Fiber | null = returnFiber.alternate?.child || null;
let lastPlacedIndex = 0;
let newIdx = 0;
```

这些变量分别表示：

```txt
resultingFirstChild   新子链表的第一个 Fiber
previousNewFiber      上一个新 Fiber，用来连接 sibling
oldFiber              当前正在比较的旧 Fiber
lastPlacedIndex       最后一个不需要移动的旧节点位置
newIdx                当前新 children 的下标
```

构建新链表靠的是：

```ts
if (previousNewFiber === null) {
  resultingFirstChild = newFiber;
} else {
  previousNewFiber.sibling = newFiber;
}

previousNewFiber = newFiber;
```

最后返回：

```ts
return resultingFirstChild;
```

也就是新的第一个 child。

---

## 11. 第一阶段：顺序比较

第一阶段代码：

```ts
for (; oldFiber !== null && newIdx < children.length; newIdx++) {
  const newFiber = updateSlot(returnFiber, oldFiber, children[newIdx]);

  if (newFiber === null) {
    break;
  }

  if (oldFiber && newFiber.alternate === null) {
    deleteChild(returnFiber, oldFiber);
  }

  lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);

  if (previousNewFiber === null) {
    resultingFirstChild = newFiber;
  } else {
    previousNewFiber.sibling = newFiber;
  }

  previousNewFiber = newFiber;
  oldFiber = oldFiber?.sibling;
}
```

第一阶段尝试按位置一一比较：

```txt
old[0] 对比 new[0]
old[1] 对比 new[1]
old[2] 对比 new[2]
```

如果当前位置能复用，就继续。

如果不能复用，就跳出第一阶段。

---

## 12. updateSlot：当前位置能不能复用

`updateSlot` 可以理解成：

> 当前位置的旧节点，能不能更新成当前位置的新节点？

源码：

```ts
function updateSlot(returnFiber: Fiber, oldFiber: Fiber, newChild: any): any {
  const key = oldFiber.key;

  if (typeof newChild === 'string' || typeof newChild === 'number') {
    if (key !== null) {
      return null;
    }
    return updateTextNode(returnFiber, oldFiber, newChild as string);
  }

  if (typeof newChild === 'object' && newChild !== null) {
    switch (newChild.$$typeof) {
      case REACT_ELEMENT_TYPE: {
        if (newChild.key === key) {
          return updateElement(returnFiber, oldFiber, newChild);
        } else {
          return null;
        }
      }
    }
  }
}
```

它先比较 `key`。

如果 `key` 不同：

```ts
return null;
```

这表示当前位置不能复用。

如果 `key` 相同：

```ts
return updateElement(returnFiber, oldFiber, newChild);
```

再进入 `type` 判断。

---

## 13. updateElement：type 能不能复用

源码：

```ts
function updateElement(returnFiber: Fiber, oldFiber: Fiber | null, element: any): Fiber {
  const elementType = element.type;

  if (oldFiber !== null) {
    if (oldFiber.elementType === elementType) {
      const existing = createWorkInProgress(oldFiber, element);
      existing.return = returnFiber;
      return existing;
    }
  }

  const created = createFiberFromElement(element);
  created.return = returnFiber;
  return created;
}
```

这里判断：

```txt
oldFiber.elementType === element.type
```

如果相同，复用旧 Fiber。

如果不同，创建新 Fiber。

注意：

```txt
key 的比较在 updateSlot
type 的比较在 updateElement
```

这两个函数合在一起，才完成“当前位置是否可复用”的判断。

---

## 14. placeChild：判断插入还是移动

`placeChild` 的作用是：

> 给新 Fiber 设置 index，并判断它需不需要打 Placement 标记。

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

这里分两种情况。

### 新节点没有 alternate

```ts
current === null
```

说明它不是从旧 Fiber 复用来的，而是新创建的。

所以需要插入 DOM：

```ts
newFiber.flags |= Placement;
```

### 新节点有 alternate

```ts
current !== null
```

说明它复用了旧 Fiber。

这时还要判断它是否需要移动。

---

## 15. lastPlacedIndex 怎么理解

`lastPlacedIndex` 可以理解成：

> 到目前为止，已经确认不需要移动的旧节点里，最靠后的旧位置。

例子：

旧：

```txt
[A, B, C, D]
```

新：

```txt
[A, C, B, D]
```

旧 index：

```txt
A: 0
B: 1
C: 2
D: 3
```

新数组从左到右处理。

处理 `A`：

```txt
oldIndex = 0
lastPlacedIndex = 0
0 < 0 ? false
不移动
lastPlacedIndex = 0
```

处理 `C`：

```txt
oldIndex = 2
lastPlacedIndex = 0
2 < 0 ? false
不移动
lastPlacedIndex = 2
```

处理 `B`：

```txt
oldIndex = 1
lastPlacedIndex = 2
1 < 2 ? true
B 需要移动
lastPlacedIndex 仍然是 2
```

处理 `D`：

```txt
oldIndex = 3
lastPlacedIndex = 2
3 < 2 ? false
不移动
lastPlacedIndex = 3
```

所以结果是：

```txt
B 打 Placement
```

这里的关键是：

```txt
如果一个旧节点的 oldIndex 小于 lastPlacedIndex
说明它在旧数组里出现在一个已经确认稳定节点的前面
但在新数组里却出现在这个稳定节点的后面
所以它需要移动
```

---

## 16. 第二阶段：新数组已经遍历完

第一阶段结束后，如果：

```ts
newIdx === children.length
```

说明新 children 已经处理完了。

那旧链表里剩下的节点都不需要了。

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

第一阶段处理完 `A`、`B` 后：

```txt
newIdx 到头了
oldFiber 还指向 C
```

于是删除剩余旧节点：

```txt
删除 C
```

---

## 17. 第二阶段：旧数组已经遍历完

如果：

```ts
oldFiber === null
```

说明旧 Fiber 已经用完了。

那新 children 剩下的节点都是新增节点。

源码：

```ts
if (oldFiber === null) {
  for (; newIdx < children.length; newIdx++) {
    const newFiber = typeof children[newIdx] === 'string' ||
      typeof children[newIdx] === 'number'
        ? createFiberFromText(children)
        : createFiberFromElement(children[newIdx]);

    newFiber.return = returnFiber;
    lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);

    if (previousNewFiber === null) {
      resultingFirstChild = newFiber;
    } else {
      previousNewFiber.sibling = newFiber;
    }

    previousNewFiber = newFiber;
  }
}
```

例子：

```txt
旧：[A, B]
新：[A, B, C, D]
```

第一阶段处理完 `A`、`B` 后：

```txt
oldFiber === null
newIdx 指向 C
```

剩下的 `C`、`D` 都是新节点。

所以它们都会打：

```txt
Placement
```

---

## 18. 第三阶段：Map 查找

如果第一阶段顺序比较断了，而且新旧两边都还有剩余节点，就进入第三阶段。

例子：

```txt
旧：[A, B, C, D]
新：[A, C, B, E]
```

第一阶段：

```txt
A 对 A，可以复用
B 对 C，key 不同，顺序断了
```

这时不能继续按位置比较。

于是把剩余旧节点放进 Map：

```ts
const existingChildren = mapRemainingChildren(oldFiber);
```

源码：

```ts
function mapRemainingChildren(currentFirstChild: Fiber | null): Map<string | number, Fiber> {
  const existingChildren = new Map();
  let child = currentFirstChild;

  while (child !== null) {
    existingChildren.set(child.key === null ? child.index : child.key, child);
    child = child.sibling;
  }

  return existingChildren;
}
```

Map 的 key 是：

```txt
有 key    用 key
无 key    用 index
```

---

## 19. updateFromMap：从 Map 里找旧节点

源码：

```ts
function updateFromMap(
  existingChildren: Map<string | number, Fiber>,
  returnFiber: Fiber,
  newIdx: number,
  newChild: any
): any {
  if (typeof newChild === 'string' || typeof newChild === 'number') {
    const matchedFiber = existingChildren.get('' + newIdx) || null;
    return updateTextNode(returnFiber, matchedFiber, newChild as string);
  }

  if (typeof newChild === 'object' && newChild !== null) {
    switch (newChild.$$typeof) {
      case REACT_ELEMENT_TYPE: {
        const matchedFiber: Fiber | null =
          existingChildren.get(newChild.key === null ? newIdx : newChild.key) || null;
        return updateElement(returnFiber, matchedFiber, newChild);
      }
    }
  }
}
```

查找规则还是：

```txt
有 key    用 key 查
无 key    用 newIdx 查
```

找到旧 Fiber 后，再通过 `updateElement` 判断 `type` 能不能复用。

如果能复用：

```txt
newFiber.alternate !== null
```

然后把这个旧节点从 Map 删除：

```ts
existingChildren.delete(newFiber.key === null ? newIdx : newFiber.key);
```

因为它已经被复用了。

---

## 20. Map 剩余节点就是删除节点

第三阶段处理完所有新 children 后：

```ts
existingChildren.forEach((child: Fiber) => deleteChild(returnFiber, child));
```

Map 里还剩下的旧节点，说明：

```txt
新 children 里没有对应节点
```

所以它们都要删除。

例子：

```txt
旧：[A, B, C, D]
新：[A, C, B, E]
```

第三阶段 Map 初始：

```txt
B -> old B
C -> old C
D -> old D
```

处理新 `C`：

```txt
找到 old C，复用
Map 删除 C
```

处理新 `B`：

```txt
找到 old B，复用
Map 删除 B
```

处理新 `E`：

```txt
Map 找不到 E，创建新 Fiber，打 Placement
```

最后 Map 剩下：

```txt
D
```

所以删除 `D`。

---

## 21. deleteChild：删除不是打在自己身上

源码：

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

这里要特别注意：

```txt
删除标记 ChildDeletion 是打在父 Fiber 上
不是打在被删除的 child 上
```

原因是：

```txt
被删除的 child 不会出现在新的 workInProgress 子链表中
如果把删除标记打在被删除节点自己身上，commit 阶段遍历新树时可能找不到它
```

所以 React 把要删除的旧节点存在父 Fiber 的 `deletions` 数组里。

结构是：

```txt
parent Fiber
  flags: ChildDeletion
  deletions: [old child A, old child B]
```

commit 阶段处理父 Fiber 时，再从 `deletions` 里真正删除 DOM。

---

## 22. Diff 和 Flags 的关系

diff 阶段主要产生三类 mutation 标记：

```ts
export const Placement = 0b0000010;
export const Update = 0b0000100;
export const ChildDeletion = 0b0001000;

export const MutationMask = Placement | Update | ChildDeletion;
```

其中 diff 阶段主要负责：

```txt
Placement       新增或移动
ChildDeletion   删除旧节点
```

而 `Update` 主要在 `completeWork` 阶段根据 props/text 是否变化产生。

所以可以这样记：

```txt
ChildFiber / diff       负责结构变化：插入、移动、删除
CompleteWork            负责内容变化：属性、文本更新
CommitWork              负责消费 flags，操作真实 DOM
```

---

## 23. Diff 不直接操作 DOM

diff 阶段不会执行：

```ts
appendChild()
insertBefore()
removeChild()
setAttribute()
```

它只是构建新 Fiber 链表，并打标记：

```txt
这个节点要插入
这个节点要移动
这个旧节点要删除
```

真正操作 DOM 在 commit 阶段。

完整链路是：

```txt
beginWork
  ↓
reconcileChildFibers
  ↓
diff 新旧 children
  ↓
构建 workInProgress 子链表
  ↓
打 Placement / ChildDeletion
  ↓
completeWork
  ↓
打 Update，冒泡 subtreeFlags
  ↓
commit
  ↓
根据 flags 操作真实 DOM
```

---

## 24. 一个完整例子

旧：

```tsx
<ul>
  <li key="a">A</li>
  <li key="b">B</li>
  <li key="c">C</li>
</ul>
```

新：

```tsx
<ul>
  <li key="b">B</li>
  <li key="a">A</li>
  <li key="d">D</li>
</ul>
```

### 第一步：第一阶段顺序比较

比较：

```txt
old a vs new b
```

`key` 不同，第一阶段结束。

### 第二步：建立 Map

旧节点都还剩着：

```txt
a -> old a
b -> old b
c -> old c
```

### 第三步：处理新 b

```txt
Map 找到 old b
type 相同，复用
oldIndex = 1
lastPlacedIndex = 0
1 < 0 ? false
b 不移动
lastPlacedIndex = 1
```

### 第四步：处理新 a

```txt
Map 找到 old a
type 相同，复用
oldIndex = 0
lastPlacedIndex = 1
0 < 1 ? true
a 打 Placement，需要移动
lastPlacedIndex 仍然是 1
```

### 第五步：处理新 d

```txt
Map 找不到 d
创建新 Fiber
d 打 Placement，需要插入
```

### 第六步：Map 剩余节点删除

Map 还剩：

```txt
c
```

所以：

```txt
c 加入 parent.deletions
parent.flags |= ChildDeletion
```

最终结果：

```txt
b 复用，不移动
a 复用，Placement
d 新建，Placement
c 删除，记录到父 Fiber.deletions
```

---

## 25. 复习口诀

可以把 diff 记成下面这几句：

```txt
先看 key，再看 type。
key 不同，当前位置不能复用。
key 相同 type 相同，复用旧 Fiber。
key 相同 type 不同，删旧建新。

数组先顺序比较。
顺序断了再建 Map。
Map 找得到就复用。
Map 找不到就新建。
Map 最后剩下的都删除。

Placement 表示插入或移动。
ChildDeletion 打在父 Fiber 上。
Update 不主要由 diff 产生，而是在 completeWork 里比较 props/text。
```

---

## 26. 当前代码里的注意点

复习时可以先按主流程理解，后面再回来校正这些细节：

### `updateElement` 复用时传参

当前代码：

```ts
const existing = createWorkInProgress(oldFiber, element);
```

更合理的是传新 props：

```ts
const existing = createWorkInProgress(oldFiber, element.props);
```

因为 Fiber 的 `pendingProps` 应该是组件 props，而不是整个 ReactElement。

### 新文本节点创建

当前数组快速路径里：

```ts
createFiberFromText(children)
```

这里传入的是整个 children 数组。

更合理的是：

```ts
createFiberFromText(children[newIdx])
```

### 文本 Map 查找

当前文本节点从 Map 查找：

```ts
existingChildren.get('' + newIdx)
```

但 `mapRemainingChildren` 里无 key 节点使用的是数字 index：

```ts
existingChildren.set(child.key === null ? child.index : child.key, child);
```

所以这里可以统一成：

```ts
existingChildren.get(newIdx)
```

这些细节不影响先理解 diff 的大结构，但后面修代码时可以逐个处理。

---

## 27. 本讲小结

这一讲重点是：

- diff 的入口是 `reconcileChildFibers`
- 单节点 diff 主要比较 `key` 和 `type`
- 数组 diff 分为顺序比较、快速路径、Map 查找
- `createWorkInProgress` 表示复用旧 Fiber
- `Placement` 表示新增或移动
- `ChildDeletion` 打在父 Fiber 上，并通过 `deletions` 保存被删除节点
- diff 只负责生成新 Fiber 链表和 flags，不直接操作 DOM

一句话总结：

> diff 的本质，是在新 children 和旧 Fiber 子链表之间找可复用节点，并把不能复用或位置变化的地方标记出来。


