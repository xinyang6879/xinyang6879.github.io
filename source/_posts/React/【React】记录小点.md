---
title: 记录
categories: react
tag: react
---

## useMemo 和 React.memo 优化组件的区别

react 中，如果父组件的状态更新了，那么也会导致子组件也会重新渲染，从头开始渲染一次子组件（若无其他优化手段的情况下

useMemo 和 React.memo 都可以用于缓存组件，当父组件的状态变化时，如果不涉及到其子组件对应条件修改时，对应的子组件是会重新跳过渲染的

React.memo 缓存的是组件渲染结果，useMemo 缓存的是计算结果。

- React.memo

  包装组件，在 props 未变化时跳过重新渲染

  ```jsx
  const MemoizedComponent = React.memo(
    function MyComponent(props) {
      /* 使用 props 渲染 */
    },
    /* 可选的自定义比较函数 */
    (prevProps, nextProps) => {
      // 返回 true 表示 props 相等（跳过渲染）
      // 返回 false 表示 props 不同（需要渲染）
      return prevProps.value === nextProps.value;
    }
  );
  ```

- useMemo

  缓存计算结果，只有当依赖项变化时才重新计算

  ```jsx
  const memoizedValue = useMemo(
    () => <Text>{a}</Text>,
    [a, b] // 依赖项数组
  );
  ```
- 对比

    | 特性	|React.memo	|useMemo|
    | ------ | ------ | ------ |
    | 本质	 | 缓存 JSX 元素|	包装组件|
    | 类型	|高阶组件 (HOC)|	React Hook|
    | 作用对象	| 整个组件	|单个值/计算结果|
    | 优化级别	| 父组件内部优化|	子组件自身优化|
    | 主要用途	| 防止不必要的组件重新渲染	|避免昂贵的重复计算|
    | 触发条件	| 当 props 改变时	|当依赖项改变时|
    | 返回值	| 记忆化组件	|记忆化值|
    | 使用位置	| 组件定义处	|组件内部|
    | 优化级别	| 组件级别	|计算级别|
    | 重新执行是否状态重置| ❌（不会导致状态重置 | ✔ |
    | 影响范围	|仅影响当前使用位置|	影响所有使用该组件的地方|

- 场景推荐
    - useMemo
        - 避免大规模组件树重复渲染
        - 保持组件引用稳定
        - 条件渲染优化
    - React.memo
        - 纯展示组件优化
        - 防止props未变的重新渲染
        - 自定义比较逻辑

    | 场景	|推荐方式|	原因|
    | ------ | ------| ------|
    | 纯展示组件	|React.memo	|保持状态，响应props变化|
    | 大型列表项渲染	|useMemo(() => <>)|避免重复创建大量元素|
    | 需要保持内部状态的组件	|React.memo	|防止状态意外重置|
    | 需要稳定引用的组件（如动画）|	useMemo(() => <>)|	确保组件引用不变|
    | 条件渲染的昂贵组件	|useMemo(() => <>)	|避免重复挂载/卸载开销|
    | 需要深度比较props的组件	|React.memo + 比较函数	|自定义比较逻辑更灵活|
    

    优化组件的创建过程时 → 使用 useMemo(() => <Component />)

    想优化组件的渲染行为时 → 使用 React.memo(Component)

- useMemo会导致状态重置

``` ts
// React 内部伪代码
function reconcileChildren(parentFiber) {
  if (parentFiber.alternate === null) {
    // 首次渲染：挂载新组件
    const newFiber = createFiberFromElement(element);
    parentFiber.child = newFiber;
  } else {
    // 更新：比较新旧元素
    const oldFiber = parentFiber.alternate.child;
    
    if (oldFiber.elementType === element.type) {
      // 类型相同：复用 Fiber（保持状态）
      reuseFiber(oldFiber, element);
    } else {
      // 类型不同：创建新 Fiber（重置状态）
      const newFiber = createFiberFromElement(element);
      parentFiber.child = newFiber;
    }
  }
}
```

- 当依赖项变化时，返回的是新 React 元素
- React 比较新旧元素时发现 element.type 相同但 element !== oldFiber.element
- React 将其视为新组件实例，创建新 Fiber 节点 → 状态重置

