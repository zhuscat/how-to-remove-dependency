---
theme: default
class: text-center
highlighter: shiki
lineNumbers: false
drawings:
  persist: false
transition: slide-left
title: 超越 99% 开发者的 React 技巧居然在？
mdc: true
---

# 超越 99% 开发者的 React 技巧竟然隐藏在？

注：标题党

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
layout: default
---

# React 重新渲染会重新执行函数组件

没有魔法，就是 JavaScript。React 框架设计思想，view = f(state)

```js {all|5-8|7|1|2|3|10-15|all}
function Counter() {
  const [count, setCount] = useState(0)
  let foo = 0

  function handleClick() {
    setCount(count + 1)
    foo++
  }

  return (
    <>
      <div>{count}</div>
      <button onClick={handleClick}>+1</button>
    </>
  )
}
```

<!--
Here is another comment.
-->

---
layout: default
---

# 比较 Vue

```js
const count = ref(0)

let foo = 0

function handleClick() {
  count.value++
  foo++
}
```

Vue 的 setup 在组件整个生命周期中只会执行一遍，这是不同的心智模型

---
layout: default
---

# `react-hooks/exhaustive-deps` 

一条折磨人的规则

```jsx
const [showAll, setShowAll] = useState(false)
const [result, setResult] = useState(null)

useEffect(() => {
  doQuery({ showAll }).then(res => { setResult(res) })
}, []) // missing dependency: showAll

function handleSearch() {
  // ...
}

return (
  <>
    <Switch checked={showAll} onChange={setShowAll} />
    <Button onClick={handleSearch}>搜索</Button>
    <Result result={result} />
  </>
)
```

---
layout: default
---

# 解法

React 团队也意识到这个问题很常见，因此在 React 库中添加了一个新的 Hook（还没有成为正式 API），这就是：

# useEffectEvent

```js
const doInitialQuery = useEffectEvent(() => {
  doQuery({ showAll }).then(res => { setResult(res) })
})

useEffect(() => {
  doInitialQuery()
}, []) // effect event 不需要放到依赖数组里
```

<div class="text-gray-400 mt-8 text-xs">
这类场景，在实战中，我们应该使用类似 React Query 或者 swc 的库
</div>

---
layout: default
---

# 实战 1

如何实现 `useEffectEvent`？`useEffectEvent` 有如下特性：

- 规范要求 `useEffectEvent` 包裹的函数只能在 `useEffect` 中使用
- 不能作为参数传递给子组件
- 被调用的时候，函数中引用的变量总是最新值

#### Tips:

- 官方目前的实现，两次渲染 `useEffectEvent` 返回的引用是不同（思考为什么），我们自己的实现可以写一个引用相同的（因为在现行 hooks 规则下，需要把他放到 deps 数组里面）
- 可以利用 `useMemo`、`useRef`、`useInsertionEffect` 等 Hook 实现，方案不唯一
- 自己的实现，是做不到跟官方实现完全相同的，因为官方的实现修改了 React 框架内部，并不是原有 Hook 的组合，我们做到功能近似即可

https://codesandbox.io/s/nice-tdd-nmlxzn

---
layout: default
---

> **Notice that you can’t “choose” your dependencies.** You will get a lint error if the dependencies you specified don’t match what React expects based on the code inside your Effect. This helps catch many bugs in your code. If you don’t want some code to re-run, [_edit the Effect code itself_ to not “need” that dependency.](https://react.dev/learn/lifecycle-of-reactive-effects#what-to-do-when-you-dont-want-to-re-synchronize)

---
layout: default
---

# 实战 2

现在我们拥有了解决问题的强有力武器，尝试解决该问题：

https://codesandbox.io/s/kind-wildflower-ghz6yc

---
layout: default
---

# 讨论

1. 我们应该将 `useEffect` 中调用的函数都用 `useEffectEvent` 都包装一下吗？
2. 为什么官方规范中，`useEffectEvent` 返回的函数只让用在 `useEffect` 中，并且不让传递给子组件
3. 为什么我们应该严格遵守 React Hook Rules

---
layout: default
---

# 最后

https://react.dev/learn/escape-hatches

超越 99% 开发者的 React 技巧，就在官方文档