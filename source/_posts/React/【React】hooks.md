---
title: hooks
categories: react
tag: react
---

- 缓存相关

    - useEffect/useLayoutEffect 
    - useCallback
    - useMemo

- react相关

    - useContext
    - useDebugValue
    - useId
    - useImperativeHandle
    - useReducer
    - useSyncExternalStore

- 数据相关

    - useDeferredValue
    - useRef
    - useState
    - useTransition 

## useCallback

### 用法

- 参数：`useCallback(fn, dependencies)`

    - `fn: function`: 缓存的函数。此函数可以接受任何参数并且返回任何值。React 将会在初次渲染而非调用时返回该函数。当进行下一次渲染时，如果 dependencies 相比于上一次渲染时没有改变，那么 React 将会返回相同的函数。否则，React 将返回在最新一次渲染中传入的函数，并且将其缓存以便之后使用。React 不会调用此函数，而是返回此函数。你可以自己决定何时调用以及是否调用。
    - `dependencies: array`: 依赖项，当依赖项发生变化时，fn会重新执行。响应式值包括 props、state，和所有在你组件内部直接声明的变量和函数。如果你的代码检查工具 配置了 React，那么它将校验每一个正确指定为依赖的响应式值。依赖列表必须具有确切数量的项，并且必须像 [dep1, dep2, dep3] 这样编写。React 使用 `Object.is` 比较每一个依赖和它的之前的值。

    - 返回值：`function`: 缓存的函数在初次渲染时，useCallback 返回你已经传入的 fn 函数。在之后的渲染中, 如果依赖没有改变，useCallback 返回上一次渲染中缓存的 fn 函数；否则返回这一次渲染传入的 fn。

``` ts

```