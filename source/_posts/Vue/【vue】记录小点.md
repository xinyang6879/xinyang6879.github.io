---
title: 【vue】记录小点
categories: vue
---

### v-model 语法糖

在对于原生组件的 v-model 中，编译时会转换为两个部分

- 一个绑定到元素的 value/check 属性，这样数据变化时会让页面更新
- 一个监听元素的变化事件（input 等），当事件触发时就改变数据

`<input v-model="message">` 会被转换成：

```javascript
_createVNode("input", {
  modelValue: message, // 属性绑定
  "onUpdate:modelValue": ($event) => (message = $event), // 事件监听
});
```

- 当初始化渲染时，将`message`的值赋给 input 的 value，所以输入框显示的是 message 的初始值。
- 当用户输入时，触发 input 事件，事件处理函数会将输入框的新值赋给`message`。
- 由于`message`是响应式数据（比如通过 ref 或 reactive 创建），它的改变会触发依赖更新，即重新执行组件的渲染函数（重新生成虚拟 DOM，然后进行 Diff 更新视图）。但是注意，这里因为 input 的 value 绑定的是 message，所以重新渲染时会设置 input 的 value，但用户正在输入时设置 value 会不会导致问题呢？不会，因为 Vue 在更新 DOM 时，如果发现是输入元素，并且它的值没有变化（或者即使变化了，但不会打断用户的输入，因为 Vue 通过异步更新和事件处理已经做了协调），所以不会干扰用户输入。

Vue 的处理机制包括：

- 输入状态检测

  - Vue 会检测到 input 元素是否处于"正在输入"状态
  - 在用户输入期间，即使组件重新渲染，也不会强制更新 input 的 value
  - 异步更新队列

- 所有 DOM 更新都会放入异步队列中

  - 用户输入期间的更新会被延迟执行，避免打断用户操作
  - DOM 属性更新协调

- Vue 在更新 input 的 value 时会检查：
  - 元素是否为当前活跃的输入元素
  - 用户是否正在进行输入操作
  - 新值是否与当前用户输入的内容冲突

### diff

c1: 旧节点；c2: 新节点；e1：旧节点的长度；e2：新节点的长度；

- 先遍历相同头节点的情况

  这里得到 i：新旧节点相同头部的 index

  ```js
  // 1. sync from start
  // (a b) c
  // (a b) d e
  while (i <= e1 && i <= e2) {
    const n1 = c1[i];
    const n2 = (c2[i] = optimized
      ? cloneIfMounted(c2[i])
      : normalizeVNode(c2[i]));
    // 如果是相同头节点
    if (isSameVNodeType(n1, n2)) {
      patch(
        n1,
        n2,
        container,
        null,
        parentComponent,
        parentSuspense,
        isSVG,
        slotScopeIds,
        optimized
      );
    } else {
      break;
    }
    i++;
  }
  ```

- 遍历尾节点

  ```js
  // 2. sync from end
  // a (b c)
  // d e (b c)
  while (i <= e1 && i <= e2) {
    const n1 = c1[e1];
    const n2 = (c2[e2] = optimized
      ? cloneIfMounted(c2[e2])
      : normalizeVNode(c2[e2]));
    if (isSameVNodeType(n1, n2)) {
      patch(
        n1,
        n2,
        container,
        null,
        parentComponent,
        parentSuspense,
        isSVG,
        slotScopeIds,
        optimized
      );
    } else {
      break;
    }
    e1--;
    e2--;
  }
  ```

- 旧节点被遍历结束了，但是新节点没有遍历结束

  这个时候剩余的新节点一定需要重新生成 dom

  ```js
  // 3. common sequence + mount
  // (a b)
  // (a b) c
  // i = 2, e1 = 1, e2 = 2
  // (a b)
  // c (a b)
  // i = 0, e1 = -1, e2 = 0
  if (i > e1) {
    if (i <= e2) {
      const nextPos = e2 + 1;
      const anchor = nextPos < l2 ? c2[nextPos].el : parentAnchor;
      while (i <= e2) {
        patch(
          null,
          (c2[i] = optimized ? cloneIfMounted(c2[i]) : normalizeVNode(c2[i])),
          container,
          anchor,
          parentComponent,
          parentSuspense,
          isSVG,
          slotScopeIds,
          optimized
        );
        i++;
      }
    }
  }
  ```

- 旧节点没有遍历结束，但是新节点遍历结束了

  那么直接把旧节点的 dom 全部卸载掉

  ```js
  // 4. common sequence + unmount
        // (a b) c
        // (a b)
        // i = 2, e1 = 2, e2 = 1
        // a (b c)
        // (b c)
        // i = 0, e1 = 0, e2 = -1
        else if (i > e2) {
            while (i <= e1) {
                unmount(c1[i], parentComponent, parentSuspense, true);
                i++;
            }
        }
  ```

- 建立剩余的新节点的节点与节点 key 的 map

  - 如果启用了优化（optimized 为 true），则使用 cloneIfMounted 复用已挂载的节点

    - 判断逻辑：
      child.el === null: 节点未挂载

      当一个 vnode 刚创建但尚未渲染到 DOM 时，el 属性为 null
      这种情况下直接返回原节点，无需克隆
      child.memo: 节点被标记为需要记忆

      如果节点有 memo 标记（用于优化），也直接返回原节点
      其他情况: 节点已挂载

      如果 child.el 不为 null（说明已经挂载到真实 DOM 元素）
      则调用 cloneVNode(child) 创建新节点

    - 工作原理：
      未挂载节点: vnode.el = null，表示该虚拟节点还未对应真实 DOM 元素

      已挂载节点: vnode.el 指向实际的 DOM 元素，表示该虚拟节点已经渲染到页面上

  - 否则使用 normalizeVNode 标准化节点
  - 同时将处理后的节点重新赋值给 c2[i]

  ```js
  const s2 = i; // next starting index
  // 5.1 build key:index map for newChildren
  const keyToNewIndexMap = new Map();
  for (i = s2; i <= e2; i++) {
    const nextChild = (c2[i] = optimized
      ? cloneIfMounted(c2[i])
      : normalizeVNode(c2[i]));
    if (nextChild.key != null) {
      if (keyToNewIndexMap.has(nextChild.key)) {
        warn$1(
          `Duplicate keys found during update:`,
          JSON.stringify(nextChild.key),
          `Make sure keys are unique.`
        );
      }
      keyToNewIndexMap.set(nextChild.key, i);
    }
  }
  ```

- 遍历旧节点，查找旧节点剩余的节点是否在新节点里面有，如果有就加上，如果没有就卸载

  ```js
  let j;
  let patched = 0;
  const toBePatched = e2 - s2 + 1;
  let moved = false;
  // used to track whether any node has moved
  let maxNewIndexSoFar = 0;
  // works as Map<newIndex, oldIndex>
  // Note that oldIndex is offset by +1
  // and oldIndex = 0 is a special value indicating the new node has
  // no corresponding old node.
  // used for determining longest stable subsequence
  const newIndexToOldIndexMap = new Array(toBePatched);
  for (i = 0; i < toBePatched; i++) newIndexToOldIndexMap[i] = 0;
  for (i = s1; i <= e1; i++) {
    const prevChild = c1[i];
    if (patched >= toBePatched) {
      // 所有新节点都已处理，剩余的旧节点需要删除
      unmount(prevChild, parentComponent, parentSuspense, true);
      continue;
    }
    let newIndex;
    if (prevChild.key != null) {
      // 有 key 的节点，直接通过 Map 查找新位置
      newIndex = keyToNewIndexMap.get(prevChild.key);
    } else {
      // 无 key 节点，需要遍历查找相同类型的节点
      for (j = s2; j <= e2; j++) {
        if (
          newIndexToOldIndexMap[j - s2] === 0 &&
          isSameVNodeType(prevChild, c2[j])
        ) {
          newIndex = j;
          break;
        }
      }
    }
    if (newIndex === undefined) {
      // 节点在新列表中不存在，需要删除
      unmount(prevChild, parentComponent, parentSuspense, true);
    } else {
      // 节点存在，记录映射关系并更新
      newIndexToOldIndexMap[newIndex - s2] = i + 1;
      if (newIndex >= maxNewIndexSoFar) {
        maxNewIndexSoFar = newIndex;
      } else {
        moved = true;
      }
      // 更新已存在的节点，prevChild：旧节点，c2[newIndex]：新节点
      patch(
        prevChild,
        c2[newIndex],
        container,
        null,
        parentComponent,
        parentSuspense,
        isSVG,
        slotScopeIds,
        optimized
      );
      patched++;
    }
  }
  ```

- 计算最长递增子序列

  - 找出新旧节点中相对位置不变的节点序列
  - 如果节点发生了移动（moved 为 true），则调用 getSequence 计算 newIndexToOldIndexMap 的最长递增子序列
  - 最长递增子序列（LIS）用于找出不需要移动的节点， 只移动位置发生变化的节点，从而最小化 DOM 操作

  ```js
  const increasingNewIndexSequence = moved
    ? getSequence(newIndexToOldIndexMap)
    : EMPTY_ARR;
  ```

- 反向遍历剩余的新节点，因为在前几个步骤中，新节点的前后都可能已经又其他节点的，通过锚点可以利用`insertBefore(newNode, referenceNode)`具体插入在哪个元素前。如果从头开始，后节点可能不存在，因此是反向遍历

  ```js
  j = increasingNewIndexSequence.length - 1;
  // looping backwards so that we can use last patched node as anchor
  for (i = toBePatched - 1; i >= 0; i--) {
    // s2新子节点数组的起始索引，即在此下标之前的节点都已经挂载好了
    const nextIndex = s2 + i;
    const nextChild = c2[nextIndex];
    // 这里就是获取当前节点的下一个节点，用于节点插入
    const anchor = nextIndex + 1 < l2 ? c2[nextIndex + 1].el : parentAnchor;
    // 如果新节点在旧节点中不存在，那么直接插入即可
    if (newIndexToOldIndexMap[i] === 0) {
      // mount new
      patch(
        null,
        nextChild,
        container,
        anchor,
        parentComponent,
        parentSuspense,
        isSVG,
        slotScopeIds,
        optimized
      );
    } else if (moved) {
      // move if:
      // There is no stable subsequence (e.g. a reverse)
      // OR current node is not among the stable sequence
      // 移动旧节点到新节点
      if (j < 0 || i !== increasingNewIndexSequence[j]) {
        move(nextChild, container, anchor, 2 /* REORDER */);
      } else {
        j--;
      }
    }
  }
  ```

- 为什么采用最长递增子序列算法进行移动，而不是把所有已存在节点进行移动

    - 采用LIS算法，只需要移动部分节点，这些节点依据LIS列表中的位置去进行定位即可
    - 移动所有节点，会耗费大量时间
    ``` 
    // 旧子节点: [A, B, C, D, E]
    // 新子节点: [B, C, E, A, D]
    // 索引映射: [1, 2, 4, 0, 3] (新索引对应的老索引)

    // 如果移动所有节点:
    // 1. B -> 移动到位置0
    // 2. C -> 移动到位置1  
    // 3. E -> 移动到位置2
    // 4. A -> 移动到位置3
    // 5. D -> 移动到位置4
    // 总共5次移动

    // 使用LIS优化:
    // LIS: [1, 2, 4] 对应 [B, C, E] - 这些节点已经按正确顺序排列
    // 需要移动: [0, 3] 对应 [A, D]
    // 1. A -> 移动到位置3
    // 2. D -> 移动到位置4  
    // 总共2次移动
    ```

    