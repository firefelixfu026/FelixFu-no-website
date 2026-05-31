# React 核心技术笔记

## 1. React 架构概述

**核心特性**：React 是用于构建用户界面的 JavaScript 库，核心目标是以组件为基本单元组织 UI，并通过状态变化驱动视图更新。React 不直接规定完整应用架构，而是专注于 View 层渲染、组件组合、状态流转与 UI 更新效率。

React 的核心机制如下：

| 核心机制 | 技术含义 | 工程价值 |
| --- | --- | --- |
| 组件化开发 | 将页面拆分为可组合、可复用、可维护的 UI 单元 | 降低复杂页面的结构耦合，提升复用与测试能力 |
| 虚拟 DOM | 使用 JavaScript 对象描述真实 DOM 结构 | 将直接 DOM 操作抽象为声明式状态更新 |
| Diff 算法 | 对比新旧虚拟 DOM 树，计算最小化更新路径 | 减少不必要的 DOM 操作，提高渲染效率 |
| 状态驱动视图 | UI 是状态的函数，状态变化触发重新渲染 | 使数据流与界面表现更可预测 |
| JSX 语法 | 在 JavaScript 中书写类 HTML 的 UI 描述 | 同时表达结构、状态、事件与组件组合关系 |

React 的典型渲染模型可以概括为：

```txt
State / Props 变化
        ↓
组件函数重新执行
        ↓
生成新的 React Element 树
        ↓
React Diff 与协调
        ↓
提交最小 DOM 更新
```

**优势对比**：

| 对比维度 | 传统原生实现 | React 范式 |
| --- | --- | --- |
| 代码结构 | HTML、CSS、JavaScript 通常按文件类型拆分，交互逻辑与 DOM 查询分散 | 以组件为边界组织结构、样式、状态与事件，业务上下文更集中 |
| DOM 操作 | 依赖 `querySelector`、`innerHTML`、`appendChild` 等命令式操作 | 通过状态变更声明 UI 结果，由 React 负责计算并提交 DOM 更新 |
| 复用性 | 复用通常依赖模板片段、函数封装或手动复制 DOM 逻辑 | 通过组件、Props、Hooks、组合模式沉淀可复用 UI 与逻辑 |

## 2. JSX 语法与组件通信基础

**严格规范**：JSX 是 JavaScript 的语法扩展，最终会被编译为 React 元素创建逻辑。JSX 看似接近 HTML，但具有更严格的结构规范与 JavaScript 表达能力。

JSX 核心红线如下：

| 规范 | 正确写法 | 说明 |
| --- | --- | --- |
| 标签必须严格闭合 | `<img src="/logo.png" alt="Logo" />` | 单标签必须自闭合，双标签必须有结束标签 |
| 组件必须返回单一根节点 | `return <><Header /><Main /></>;` | 可使用 `<>...</>` 或 `<Fragment>...</Fragment>` 包裹多个兄弟节点 |
| 自定义组件必须大写字母开头 | `<UserCard />` | 小写标签会被 React 当作原生 HTML 标签处理 |
| 使用 `className` 替代 `class` | `<div className="card" />` | `class` 是 JavaScript 保留字相关语义，React 使用 `className` 映射 DOM class |
| 使用 `htmlFor` 替代 `for` | `<label htmlFor="email">Email</label>` | `for` 在 JavaScript 中有语法含义，JSX 使用 `htmlFor` |
| JSX 中嵌入表达式使用 `{}` | `<p>{user.name}</p>` | 花括号内可以放入 JavaScript 表达式，不能直接放语句 |
| 内联样式使用对象 | `<div style={{ color: "red" }} />` | CSS 属性使用驼峰命名，例如 `backgroundColor` |

```jsx
import { Fragment } from "react";

function UserPanel({ user }) {
  return (
    <Fragment>
      <h2>{user.name}</h2>
      <img src={user.avatar} alt={user.name} />
    </Fragment>
  );
}
```

**状态与事件机制**：React 函数组件的重新渲染本质是组件函数重新执行。组件内部的普通局部变量会在每次执行时重新创建，因此不能用于保存跨渲染周期的 UI 状态。需要触发视图更新并跨渲染保留的数据，应使用 `useState`、`useReducer`、外部状态库或父组件传入的 Props。

事件绑定使用驼峰命名，并接收函数引用，而不是立即执行函数调用结果。

```jsx
function CounterButton() {
  const handleClick = () => {
    console.log("clicked");
  };

  return <button onClick={handleClick}>增加</button>;
}
```

常见事件规范如下：

| 原生 HTML 写法 | React JSX 写法 | 说明 |
| --- | --- | --- |
| `onclick="handleClick()"` | `onClick={handleClick}` | 传入函数引用 |
| `oninput="handleInput()"` | `onInput={handleInput}` | 事件名使用驼峰 |
| `onchange="handleChange()"` | `onChange={handleChange}` | 表单场景中高频使用 |
| `onsubmit="handleSubmit()"` | `onSubmit={handleSubmit}` | 通常需要调用 `event.preventDefault()` 阻止默认提交 |

**Props 参数传递**：

Props 是父组件传递给子组件的只读输入。React 的数据流默认是单向的：父组件通过 Props 向下传递数据，子组件通过调用父组件传入的回调函数向上传递事件或意图。

Props 的关键规则如下：

| 规则 | 说明 |
| --- | --- |
| 只读不可变 | 子组件不应直接修改 Props，否则会破坏数据流可预测性 |
| 父到子单向流动 | 数据来源由父组件或上层状态控制，子组件只消费输入 |
| 更新由父组件驱动 | 子组件需要修改父级数据时，应调用父组件传入的事件处理函数 |
| 可通过解构提升可读性 | 函数组件参数中直接解构 Props，减少重复访问 |
| 可通过展开语法透传 | 使用 `...props` 将剩余参数传递给底层元素或子组件 |

对象解构示例：

```jsx
function UserCard({ name, title, avatarUrl }) {
  return (
    <article className="user-card">
      <img src={avatarUrl} alt={name} />
      <h3>{name}</h3>
      <p>{title}</p>
    </article>
  );
}

function App() {
  return (
    <UserCard
      name="Ada Lovelace"
      title="Mathematician"
      avatarUrl="/avatars/ada.png"
    />
  );
}
```

展开语法透传示例：

```jsx
function Button({ variant = "primary", className = "", ...props }) {
  const variantClass = variant === "primary" ? "btn-primary" : "btn-secondary";

  return (
    <button
      className={`btn ${variantClass} ${className}`}
      {...props}
    />
  );
}

function Toolbar() {
  return (
    <Button
      type="button"
      variant="primary"
      disabled={false}
      onClick={() => console.log("save")}
    >
      保存
    </Button>
  );
}
```

展开语法应避免无边界透传敏感或无效属性。公共组件中建议明确区分业务 Props 与 DOM Props，减少未知属性进入 DOM。

**列表渲染 (map)**：`array.map()` 是 React 中渲染列表的常用方式。它将数据数组映射为 React 元素数组，适合表达“数据集合 -> UI 集合”的声明式关系。

```jsx
const todos = [
  { id: "t1", text: "学习 JSX", done: true },
  { id: "t2", text: "理解 Props", done: false },
  { id: "t3", text: "掌握 Hooks", done: false },
];

function TodoList() {
  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>
          <label>
            <input type="checkbox" checked={todo.done} readOnly />
            {todo.text}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

`key` 是 React 识别同级列表元素身份的稳定标识。React 在列表更新时会根据 `key` 判断元素是新增、删除、移动还是复用。正确的 `key` 可以减少不必要的 DOM 更新，并避免组件状态错位。

`key` 使用原则如下：

| 原则 | 说明 |
| --- | --- |
| 同级唯一 | `key` 只需要在当前列表的兄弟节点之间唯一 |
| 稳定不变 | 数据顺序变化后，同一条数据仍应拥有相同 `key` |
| 优先使用业务 ID | 数据库 ID、UUID、稳定编码通常优于数组下标 |
| 避免随机值 | `Math.random()` 会导致每次渲染都被视为新元素 |
| 谨慎使用下标 | 只适合静态列表；可排序、可插入、可删除列表不应使用下标 |

错误示例：

```jsx
items.map((item, index) => (
  <TodoItem key={index} item={item} />
));
```

当列表插入、删除或排序时，数组下标会变化，React 可能错误复用旧组件实例，导致输入框内容、动画状态、焦点状态等与数据不匹配。

## 3. 工程化开发环境搭建

**脚手架对比**：

| 脚手架名称 | 构建命令 | 核心定位/优势 |
| --- | --- | --- |
| Vite | `npm create vite@latest my-app -- --template react-ts` | 面向现代前端的轻量级构建工具，开发服务器启动快，HMR 性能好，适合 SPA、组件库、后台系统等客户端应用 |
| Next.js | `npx create-next-app@latest my-app` | React 全栈框架，支持 SSR、SSG、ISR、API Routes、文件路由与 App Router，适合内容站点、SEO 场景、服务端渲染与全栈应用 |

选择建议如下：

| 场景 | 更适合的方案 | 原因 |
| --- | --- | --- |
| 纯客户端 SPA | Vite | 配置轻、启动快、构建链路直接 |
| 管理后台 | Vite | 通常不依赖 SEO，重交互和权限路由更常见 |
| 内容站点或营销页 | Next.js | SSR/SSG 更利于首屏性能与搜索引擎抓取 |
| 全栈 React 应用 | Next.js | 路由、服务端组件、接口层与部署模型更完整 |
| 组件库开发 | Vite | 库模式、开发体验与构建速度更适合轻量包开发 |

**核心文件渲染链路**：以 Vite React 项目为例，典型入口链路为 `index.html -> main.tsx -> App.tsx`。

| 文件 | 作用 | 关键职责 |
| --- | --- | --- |
| `index.html` | 浏览器入口 HTML | 提供根 DOM 节点，加载模块脚本 |
| `src/main.tsx` | React 应用入口 | 获取根节点，创建 React Root，渲染根组件 |
| `src/App.tsx` | 根组件 | 组织页面结构、路由、布局与顶层状态 |

典型结构如下：

```html
<!-- index.html -->
<!doctype html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>React App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

```tsx
// src/main.tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App";

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

```tsx
// src/App.tsx
export default function App() {
  return <main>React Application</main>;
}
```

`createRoot().render()` 的作用是创建 React 18 的并发根节点，并将 React 组件树挂载到真实 DOM 容器中。`createRoot` 替代旧版 `ReactDOM.render`，使应用具备 React 18 的自动批处理、并发渲染能力基础。

`<StrictMode>` 是开发环境辅助组件，不会渲染真实 DOM。它用于暴露潜在副作用问题、废弃 API 使用、非纯渲染逻辑等风险。在 React 18 开发环境中，严格模式可能会对部分生命周期或 Effect 执行额外检查，因此副作用逻辑必须具备幂等性与正确清理能力。

## 4. Hooks 核心机制深度剖析

**函数式编程理念**：纯函数是指在相同输入下始终返回相同输出，并且执行过程中不修改外部状态、不产生不可控副作用的函数。React 函数组件在理想模型中接近纯函数：输入为 Props 与 State，输出为 UI 描述。

函数组件的直接问题是：普通局部变量无法跨渲染保存状态，数据请求、订阅、定时器、手动 DOM 操作等副作用也不能直接写入渲染过程。Hooks 提供了在函数组件中接入 React 状态系统与副作用生命周期的能力。

Hooks 的基本约束如下：

| 规则 | 说明 |
| --- | --- |
| 只在函数组件或自定义 Hook 中调用 | 保证 Hook 与 React 渲染上下文绑定 |
| 只在顶层调用 | 不应写在条件、循环、嵌套函数中，保证每次渲染调用顺序一致 |
| Hook 名称以 `use` 开头 | 便于静态检查与语义识别 |
| 渲染逻辑保持纯净 | 数据请求、订阅、定时器等副作用应进入 `useEffect` 等 Hook |

### 4.1 useState (状态管理)

**机制解析**：`useState` 用于在函数组件中声明状态。它接收初始值，返回一个数组：第一项为当前状态值，第二项为状态更新函数。

```tsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

数组结构通常通过解构赋值使用：

| 返回项 | 示例 | 说明 |
| --- | --- | --- |
| 当前状态 | `count` | 当前渲染快照中的状态值 |
| 更新函数 | `setCount` | 通知 React 安排下一次状态更新与重新渲染 |

初始值只在组件首次渲染时使用。昂贵的初始计算应传入函数，避免每次渲染重复执行。

```tsx
const [items, setItems] = useState(() => loadInitialItems());
```

**避坑指南**：React 状态更新依赖引用变化判断。直接修改对象或数组的内存地址不会形成新的引用，可能导致 React 无法正确感知变化，也会破坏不可变数据约束。

错误示例：

```tsx
const [user, setUser] = useState({ name: "Ada", age: 28 });

function updateAge() {
  user.age = 29;
  setUser(user);
}
```

正确示例：

```tsx
const [user, setUser] = useState({ name: "Ada", age: 28 });

function updateAge() {
  setUser({
    ...user,
    age: 29,
  });
}
```

对象与数组更新规则如下：

| 数据结构 | 错误方式 | 推荐方式 |
| --- | --- | --- |
| 对象 | `user.name = "Grace"` | `setUser({ ...user, name: "Grace" })` |
| 数组追加 | `list.push(item)` | `setList([...list, item])` |
| 数组删除 | `list.splice(index, 1)` | `setList(list.filter((item) => item.id !== id))` |
| 数组替换 | `list[index] = next` | `setList(list.map((item) => item.id === id ? next : item))` |

`useState` 的更新函数默认执行替换操作，不会像 class component 的 `setState` 一样自动浅合并对象。

```tsx
const [form, setForm] = useState({
  email: "",
  password: "",
});

function updateEmail(email: string) {
  setForm({
    ...form,
    email,
  });
}
```

React 18 默认启用 Automatic Batching（自动批处理）。在同一个事件循环任务中，多次状态更新会被合并为一次渲染提交，从而减少重复渲染。

```tsx
function handleClick() {
  setCount((count) => count + 1);
  setVisible(true);
  setStatus("done");
}
```

当下一次状态依赖上一次状态时，应使用函数式更新，避免读取当前渲染快照中的旧值。

```tsx
setCount((current) => current + 1);
```

### 4.2 useEffect (副作用处理)

**参数解构**：`useEffect` 用于处理渲染完成后的副作用。典型副作用包括网络请求、事件订阅、定时器、日志上报、手动操作 DOM、与非 React 系统同步状态等。

基本语法如下：

```tsx
useEffect(() => {
  // effect: 执行副作用

  return () => {
    // cleanup: 清理副作用
  };
}, [deps]);
```

`useEffect` 的核心参数含义如下：

| 组成部分 | 说明 | 典型用途 |
| --- | --- | --- |
| `effect` 回调函数 | 浏览器完成渲染提交后执行的副作用逻辑 | 发起请求、订阅事件、启动定时器 |
| `cleanup` 清除函数 | 下一次 effect 执行前或组件卸载时执行 | 取消订阅、清除定时器、中止请求、释放资源 |
| `deps` 依赖数组 | 声明 effect 使用到的响应式值 | 控制副作用何时重新执行 |

依赖数组行为如下：

| 写法 | 执行时机 | 适用场景 |
| --- | --- | --- |
| 不传依赖数组 | 每次渲染后都执行 | 极少使用，容易造成重复副作用 |
| `[]` | 组件挂载后执行一次，卸载时清理 | 初始化订阅、首屏请求、第三方实例创建 |
| `[a, b]` | 首次渲染后执行，且 `a` 或 `b` 变化后重新执行 | 与特定状态、Props、派生参数同步 |

```tsx
useEffect(() => {
  const controller = new AbortController();

  fetch(`/api/users/${userId}`, {
    signal: controller.signal,
  });

  return () => {
    controller.abort();
  };
}, [userId]);
```

**闭包陷阱与状态快照**：函数组件每次渲染都会创建新的函数作用域。该次渲染中的 Props、State、事件处理函数与 Effect 回调共同构成一个“状态快照”。异步回调、定时器或事件监听器如果引用了某次渲染中的 state，就会持续读取该次快照中的旧值。

典型问题如下：

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      setCount(count + 1);
    }, 1000);

    return () => clearInterval(timer);
  }, []);

  return <p>{count}</p>;
}
```

该示例中，`deps` 为空数组，Effect 只在首次渲染时执行。定时器回调捕获的是首次渲染的 `count`，因此后续回调仍基于旧快照计算。

解决方案一：使用函数式更新读取最新状态。

```tsx
useEffect(() => {
  const timer = setInterval(() => {
    setCount((current) => current + 1);
  }, 1000);

  return () => clearInterval(timer);
}, []);
```

解决方案二：完整声明依赖，使 Effect 随状态变化重新同步。

```tsx
useEffect(() => {
  const timer = setInterval(() => {
    setCount(count + 1);
  }, 1000);

  return () => clearInterval(timer);
}, [count]);
```

两种方案的差异如下：

| 方案 | 优点 | 代价 |
| --- | --- | --- |
| 函数式更新 | 不依赖外部 `count`，定时器无需反复重建 | 仅适合下一状态可由上一状态推导的场景 |
| 完整依赖 | 与 React 数据流完全同步，闭包语义清晰 | 依赖变化会触发清理与重建副作用 |

状态快照并非 React 缺陷，而是 JavaScript 闭包与函数式渲染模型共同产生的结果。关键是明确每个回调读取的是哪一次渲染中的值。

**依赖项最佳实践**：Effect 依赖项应完整声明所有在 Effect 内部读取的响应式值，包括 Props、State、组件内定义的函数、派生变量等。故意省略依赖项会让 Effect 与实际数据流脱节，产生隐蔽的陈旧数据问题。

依赖声明原则如下：

| 原则 | 说明 |
| --- | --- |
| 不欺骗依赖数组 | Effect 使用了某个响应式值，就应将其加入依赖 |
| 优先重构数据流 | 不应通过删除依赖掩盖重复请求或循环更新问题 |
| 减少不稳定引用 | 对象、数组、函数在每次渲染都会创建新引用，可能触发额外 Effect |
| 清理必须完整 | 订阅、定时器、外部实例、未完成请求都应在 cleanup 中释放 |

减少不必要依赖刷新的常见方式如下：

| 方式 | 适用场景 | 示例 |
| --- | --- | --- |
| 使用函数式更新 | 下一状态只依赖上一状态 | `setCount((c) => c + 1)` |
| 使用 `useReducer` | 多个状态更新存在复杂关联 | 将状态转移逻辑收敛到 reducer |
| 将函数移出组件 | 函数不依赖组件内部响应式值 | 模块级纯函数 |
| 使用 `useCallback` | 函数需要作为依赖或传给子组件 | `const fn = useCallback(() => {}, [deps])` |
| 将对象创建放入 Effect 内部 | 对象只为副作用服务 | 避免对象引用成为外部依赖 |

`useReducer` 示例：

```tsx
import { useReducer } from "react";

type State = {
  count: number;
};

type Action =
  | { type: "increment" }
  | { type: "reset" };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "reset":
      return { count: 0 };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <button onClick={() => dispatch({ type: "increment" })}>
      {state.count}
    </button>
  );
}
```

`useCallback` 示例：

```tsx
import { useCallback, useEffect } from "react";

function SearchPanel({ keyword }: { keyword: string }) {
  const fetchResults = useCallback(async () => {
    const response = await fetch(`/api/search?q=${encodeURIComponent(keyword)}`);
    return response.json();
  }, [keyword]);

  useEffect(() => {
    fetchResults();
  }, [fetchResults]);

  return null;
}
```

`useCallback` 不是性能优化的默认答案。只有当函数引用稳定性确实影响 Effect 依赖、子组件 memo、事件订阅清理或第三方库交互时，才有明确使用价值。

## 5. 路由生态 (React Router)

**核心组件**：`react-router-dom` 是 React Web 应用中常用的路由库，用于将 URL 路径映射到对应组件，并提供导航、参数读取、嵌套路由等能力。

核心基础构成如下：

| 组件 | 作用 | 说明 |
| --- | --- | --- |
| `<BrowserRouter>` | 路由容器 | 基于 HTML5 History API 监听 URL 变化，为内部组件提供路由上下文 |
| `<Routes>` | 路由匹配器 | 在子 `<Route>` 中选择当前 URL 匹配度最高的路由分支 |
| `<Route>` | 路径与组件映射 | 使用 `path` 描述路径规则，使用 `element` 指定渲染组件 |

基础示例：

```tsx
import { BrowserRouter, Link, Route, Routes } from "react-router-dom";

function HomePage() {
  return <h1>首页</h1>;
}

function AboutPage() {
  return <h1>关于</h1>;
}

function AppRouter() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">首页</Link>
        <Link to="/about">关于</Link>
      </nav>

      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/about" element={<AboutPage />} />
      </Routes>
    </BrowserRouter>
  );
}
```

常见路由能力如下：

| 能力 | API / 组件 | 用途 |
| --- | --- | --- |
| 声明式跳转 | `<Link to="/path" />` | 替代 `<a>`，避免整页刷新 |
| 命令式跳转 | `useNavigate()` | 表单提交、登录成功后跳转 |
| 路由参数 | `useParams()` | 读取 `/users/:id` 中的动态参数 |
| 查询参数 | `useSearchParams()` | 读取和更新 URL query string |
| 嵌套路由 | `<Outlet />` | 构建布局路由、二级页面、模块化路由 |
| 兜底路由 | `<Route path="*" />` | 处理 404 或未匹配路径 |

## 6. 常见环境报错指南

- 依赖缺失报错：`Cannot find module 'react'`

  原因通常是项目依赖尚未安装，或 `node_modules` 被删除后未重新安装。进入项目根目录后执行依赖安装命令。

  ```bash
  npm install
  ```

  若使用其他包管理器，应执行对应命令：

  ```bash
  pnpm install
  yarn install
  ```

- 脚本运行权限受限：`无法加载文件 ... 因为在此系统上禁止运行脚本`

  该错误通常出现在 Windows PowerShell 执行 npm、pnpm、yarn 或脚手架命令时，原因是当前执行策略限制脚本运行。可使用管理员身份运行 PowerShell 终端，并调整执行策略。

  ```powershell
  Set-ExecutionPolicy RemoteSigned
  ```

  更保守的做法是只修改当前用户范围：

  ```powershell
  Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
  ```

  调整后重新打开终端，再执行项目命令。企业或学校设备可能受组策略限制，此时需要遵循设备管理策略或使用允许的终端环境。
