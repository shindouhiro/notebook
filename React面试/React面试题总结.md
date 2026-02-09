# React 面试题总结

> 📅 更新时间：2024年12月
> 🎯 本文档涵盖React面试中最常见的核心问题

---

## 目录
- [[#一、React 基础概念]]
- [[#二、组件与生命周期]]
- [[#三、Hooks 深入]]
- [[#四、状态管理]]
- [[#五、性能优化]]
- [[#六、React 原理]]
- [[#七、React Router]]
- [[#八、实战与最佳实践]]

---

## 一、React 基础概念

### 1.1 什么是 React？它的核心特点是什么？

**答案：**
React 是一个用于构建用户界面的 JavaScript 库，由 Facebook 开发和维护。

**核心特点：**
- **声明式编程**：描述 UI 应该是什么样子，React 负责更新 DOM
- **组件化**：将 UI 拆分成独立、可复用的组件
- **虚拟 DOM**：通过虚拟 DOM 提高性能
- **单向数据流**：数据从父组件流向子组件
- **JSX**：JavaScript 的语法扩展，可以在 JS 中写类似 HTML 的代码

---

### 1.2 什么是 JSX？为什么使用它？

**答案：**
JSX 是 JavaScript 的语法扩展，允许在 JavaScript 代码中编写类似 HTML 的标记。

```jsx
// JSX 示例
const element = <h1>Hello, {name}</h1>;

// 编译后的 JavaScript
const element = React.createElement('h1', null, 'Hello, ', name);
```

**使用 JSX 的原因：**
1. 更直观地描述 UI 结构
2. 编译时可以发现错误
3. 可以在标记中嵌入 JavaScript 表达式
4. 防止 XSS 注入攻击（自动转义）

---

### 1.3 React 中的 key 有什么作用？

**答案：**
Key 帮助 React 识别哪些元素改变了（添加、删除或重新排序）。

```jsx
// ❌ 不推荐：使用索引作为 key
{items.map((item, index) => <li key={index}>{item}</li>)}

// ✅ 推荐：使用唯一标识符
{items.map(item => <li key={item.id}>{item.name}</li>)}
```

**关键点：**
- Key 在兄弟节点中必须唯一
- 不推荐使用数组索引作为 key（除非列表是静态的）
- Key 不会传递给子组件

---

### 1.4 受控组件与非受控组件的区别？

| 特性 | 受控组件 | 非受控组件 |
|------|---------|-----------|
| 数据管理 | React state | DOM 自身 |
| 获取值方式 | state 变量 | ref |
| 表单验证 | 实时验证 | 提交时验证 |
| 推荐场景 | 需要即时反馈 | 简单表单 |

```jsx
// 受控组件
const [value, setValue] = useState('');
<input value={value} onChange={e => setValue(e.target.value)} />

// 非受控组件
const inputRef = useRef();
<input ref={inputRef} defaultValue="initial" />
```

---

## 二、组件与生命周期

### 2.1 类组件的生命周期方法

```
┌─────────────────────────────────────────────────────────┐
│                      挂载阶段                            │
├─────────────────────────────────────────────────────────┤
│  constructor() → getDerivedStateFromProps() →           │
│  render() → componentDidMount()                         │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                      更新阶段                            │
├─────────────────────────────────────────────────────────┤
│  getDerivedStateFromProps() → shouldComponentUpdate() → │
│  render() → getSnapshotBeforeUpdate() →                 │
│  componentDidUpdate()                                   │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                      卸载阶段                            │
├─────────────────────────────────────────────────────────┤
│  componentWillUnmount()                                 │
└─────────────────────────────────────────────────────────┘
```

---

### 2.2 函数组件如何模拟生命周期？

```jsx
import { useEffect } from 'react';

function MyComponent() {
  // componentDidMount
  useEffect(() => {
    console.log('组件挂载');
    
    // componentWillUnmount
    return () => {
      console.log('组件卸载');
    };
  }, []);

  // componentDidUpdate (监听特定依赖)
  useEffect(() => {
    console.log('count 更新了');
  }, [count]);

  // 每次渲染都执行
  useEffect(() => {
    console.log('每次渲染');
  });
}
```

---

### 2.3 函数组件和类组件的区别？

| 特性 | 函数组件 | 类组件 |
|------|---------|--------|
| 语法 | 简洁，只需函数 | 需要 class 和 render |
| 状态管理 | useState Hook | this.state |
| 生命周期 | useEffect Hook | 生命周期方法 |
| this 绑定 | 无需考虑 | 需要注意 |
| 性能 | 略优（无实例创建） | 稍差 |
| 代码复用 | 自定义 Hooks | HOC、Render Props |

---

## 三、Hooks 深入

### 3.1 常用 Hooks 总览

```jsx
// 1. useState - 状态管理
const [count, setCount] = useState(0);

// 2. useEffect - 副作用处理
useEffect(() => { /* 副作用 */ }, [deps]);

// 3. useContext - 上下文消费
const theme = useContext(ThemeContext);

// 4. useReducer - 复杂状态逻辑
const [state, dispatch] = useReducer(reducer, initialState);

// 5. useMemo - 计算值缓存
const memoizedValue = useMemo(() => computeExpensive(a, b), [a, b]);

// 6. useCallback - 函数缓存
const memoizedFn = useCallback(() => doSomething(a, b), [a, b]);

// 7. useRef - 引用保持
const inputRef = useRef(null);

// 8. useLayoutEffect - 同步副作用（DOM 变更后同步执行）
useLayoutEffect(() => { /* 同步执行 */ }, []);
```

---

### 3.2 useState 的原理和注意事项

**原理：**
- 通过闭包和数组实现状态保持
- 每次渲染时，按照调用顺序返回对应状态

**注意事项：**
```jsx
// ❌ 错误：直接修改状态
state.items.push(newItem);
setState(state);

// ✅ 正确：创建新对象/数组
setState(prev => ({
  ...prev,
  items: [...prev.items, newItem]
}));

// ❌ 陷阱：在循环/条件中使用
if (condition) {
  const [value, setValue] = useState(0); // 错误！
}

// 批量更新问题（React 18 自动批处理）
setCount(count + 1);
setCount(count + 1); // 仍然只加 1

setCount(c => c + 1);
setCount(c => c + 1); // 正确加 2
```

---

### 3.3 useEffect 与 useLayoutEffect 的区别

```
浏览器渲染流程：
JS执行 → 样式计算 → 布局 → 绘制 → 合成

useEffect:      [渲染完成] ─────→ [异步执行回调]
useLayoutEffect: [DOM更新] → [同步执行回调] → [渲染]
```

| useEffect | useLayoutEffect |
|-----------|-----------------|
| 异步执行 | 同步执行 |
| 不阻塞渲染 | 阻塞渲染 |
| 大多数副作用 | DOM 测量、动画 |

---

### 3.4 useMemo 和 useCallback 的区别与使用场景

```jsx
// useMemo: 缓存计算结果
const expensiveResult = useMemo(() => {
  return heavyComputation(data);
}, [data]);

// useCallback: 缓存函数引用
const handleClick = useCallback(() => {
  console.log(count);
}, [count]);
```

**使用场景：**
- `useMemo`: 避免昂贵的重复计算
- `useCallback`: 优化子组件（配合 `React.memo`）、作为其他 Hook 的依赖

---

### 3.5 自定义 Hook 的规则和示例

**规则：**
1. 必须以 `use` 开头
2. 只在 React 函数组件或自定义 Hook 中调用
3. 不能在条件语句中调用

```jsx
// 自定义 Hook 示例：useLocalStorage
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      return initialValue;
    }
  });

  const setValue = useCallback((value) => {
    try {
      const valueToStore = value instanceof Function 
        ? value(storedValue) 
        : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  }, [key, storedValue]);

  return [storedValue, setValue];
}

// 使用
const [name, setName] = useLocalStorage('name', 'Guest');
```

---

## 四、状态管理

### 4.1 React 状态管理方案对比

| 方案 | 适用场景 | 学习曲线 | 特点 |
|------|---------|---------|------|
| useState/useReducer | 组件内状态 | 低 | 简单直接 |
| Context API | 跨组件共享少量状态 | 低 | 内置支持 |
| Redux | 大型应用复杂状态 | 高 | 可预测、时间旅行 |
| MobX | 喜欢面向对象 | 中 | 响应式、简洁 |
| Zustand | 轻量级全局状态 | 低 | 简单、无样板代码 |
| Recoil | 原子化状态 | 中 | Facebook 出品、灵活 |
| Jotai | 原子化状态 | 低 | 极简、TypeScript友好 |

---

### 4.2 Context API 使用及性能优化

```jsx
// 创建 Context
const ThemeContext = createContext('light');

// Provider 提供值
function App() {
  const [theme, setTheme] = useState('light');
  
  // 优化：使用 useMemo 避免不必要的重渲染
  const value = useMemo(() => ({ theme, setTheme }), [theme]);
  
  return (
    <ThemeContext.Provider value={value}>
      <MyComponent />
    </ThemeContext.Provider>
  );
}

// 消费
function ThemedButton() {
  const { theme, setTheme } = useContext(ThemeContext);
  return <button className={theme}>Toggle</button>;
}
```

**Context 性能陷阱与解决方案：**
1. 拆分 Context（读写分离）
2. 使用 `useMemo` 包裹 value
3. 组件拆分 + `React.memo`

---

### 4.3 Redux 核心概念

```
┌────────────┐      dispatch(action)     ┌────────────┐
│            │ ─────────────────────────→ │            │
│    View    │                            │   Store    │
│            │ ←───────────────────────── │            │
└────────────┘      subscribe/state       └────────────┘
                                                │
                                                ↓
                                         ┌────────────┐
                                         │  Reducers  │
                                         └────────────┘
```

```jsx
// Action
const increment = () => ({ type: 'INCREMENT' });

// Reducer
const counterReducer = (state = 0, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1;
    default:
      return state;
  }
};

// Store
const store = createStore(counterReducer);

// React-Redux Hooks
const count = useSelector(state => state.counter);
const dispatch = useDispatch();
dispatch(increment());
```

---

## 五、性能优化

### 5.1 React 性能优化策略总览

```
                    ┌─────────────────────────────┐
                    │       性能优化策略           │
                    └─────────────────────────────┘
                                  │
          ┌───────────────────────┼───────────────────────┐
          ↓                       ↓                       ↓
   ┌─────────────┐        ┌─────────────┐        ┌─────────────┐
   │  渲染优化    │        │  计算优化    │        │  加载优化    │
   └─────────────┘        └─────────────┘        └─────────────┘
          │                       │                       │
    ├─ React.memo          ├─ useMemo            ├─ 代码分割
    ├─ shouldComponentUpdate├─ useCallback      ├─ 懒加载
    ├─ PureComponent       └─ 虚拟列表          ├─ Suspense
    └─ key 优化                                  └─ 预加载
```

---

### 5.2 React.memo 的使用

```jsx
// 基本使用
const MyComponent = React.memo(function MyComponent(props) {
  return <div>{props.name}</div>;
});

// 自定义比较函数
const MyComponent = React.memo(
  function MyComponent(props) {
    return <div>{props.data.name}</div>;
  },
  (prevProps, nextProps) => {
    // 返回 true 表示不需要重新渲染
    return prevProps.data.id === nextProps.data.id;
  }
);
```

---

### 5.3 代码分割与懒加载

```jsx
import { lazy, Suspense } from 'react';

// 懒加载组件
const HeavyComponent = lazy(() => import('./HeavyComponent'));

// 路由级别的代码分割
const routes = [
  {
    path: '/dashboard',
    component: lazy(() => import('./pages/Dashboard')),
  },
];

// 使用 Suspense 提供加载状态
function App() {
  return (
    <Suspense fallback={<Loading />}>
      <HeavyComponent />
    </Suspense>
  );
}
```

---

### 5.4 虚拟列表

当渲染大量列表项时，使用虚拟列表只渲染可视区域的项目：

```jsx
// 使用 react-window
import { FixedSizeList } from 'react-window';

function VirtualList({ items }) {
  return (
    <FixedSizeList
      height={400}
      itemCount={items.length}
      itemSize={50}
      width={300}
    >
      {({ index, style }) => (
        <div style={style}>{items[index].name}</div>
      )}
    </FixedSizeList>
  );
}
```

---

## 六、React 原理

### 6.1 虚拟 DOM 与 Diff 算法

**虚拟 DOM 是什么？**
```jsx
// JSX
<div className="container">
  <h1>Title</h1>
</div>

// 虚拟 DOM（JavaScript 对象）
{
  type: 'div',
  props: {
    className: 'container',
    children: {
      type: 'h1',
      props: { children: 'Title' }
    }
  }
}
```

**Diff 算法策略：**
1. **同层比较**：只比较同一层级的节点
2. **类型比较**：不同类型直接替换整个子树
3. **Key 优化**：通过 key 识别移动、添加、删除

```
Old Tree:     A          New Tree:     A
            / | \                    / | \
           B  C  D                  B  E  D
           
Diff 结果: C 被替换为 E，B 和 D 保持不变
```

---

### 6.2 React Fiber 架构

**为什么需要 Fiber？**
- React 15 的 Stack Reconciler 是同步递归，无法中断
- 长任务会阻塞主线程，导致页面卡顿

**Fiber 的核心特性：**
1. **增量渲染**：将渲染工作分割成多个小任务
2. **任务优先级**：不同更新有不同优先级
3. **可中断/恢复**：高优先级任务可以打断低优先级

```
┌────────────────────────────────────────────────────────┐
│                    Fiber 工作循环                       │
├────────────────────────────────────────────────────────┤
│                                                        │
│   [浏览器空闲时间]                                      │
│         ↓                                              │
│   ┌─────────────┐    ┌─────────────┐                   │
│   │ 执行工作单元 │ ←→ │  检查时间片  │                   │
│   └─────────────┘    └─────────────┘                   │
│         ↓                   ↓                          │
│   还有工作 → 继续      时间用完 → 让出控制权             │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

### 6.3 React 的批量更新（Batching）

```jsx
// React 17: 只有事件处理函数中自动批处理
function handleClick() {
  setCount(c => c + 1);  // 不会立即渲染
  setFlag(f => !f);      // 不会立即渲染
  // 批量处理，只渲染一次
}

// React 18: 自动批处理扩展到所有场景
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React 18: 也会批量处理
}, 1000);

// 如果需要强制同步更新
import { flushSync } from 'react-dom';

flushSync(() => {
  setCount(c => c + 1);
});
// DOM 已更新
console.log(document.getElementById('counter').textContent);
```

---

### 6.4 React 的调和（Reconciliation）过程

```
┌──────────────────────────────────────────────────────┐
│                    调和过程                           │
├──────────────────────────────────────────────────────┤
│                                                      │
│  1. 触发更新 (setState/forceUpdate/props变化)         │
│         ↓                                            │
│  2. 创建新的 Fiber 树 (workInProgress tree)           │
│         ↓                                            │
│  3. Diff 比较 (current tree vs workInProgress tree)  │
│         ↓                                            │
│  4. 标记需要更新的节点 (effectTag)                     │
│         ↓                                            │
│  5. Commit 阶段：应用更新到 DOM                        │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 七、React Router

### 7.1 React Router v6 核心 API

```jsx
import {
  BrowserRouter,
  Routes,
  Route,
  Link,
  useNavigate,
  useParams,
  useSearchParams,
  Outlet,
} from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Layout />}>
          <Route index element={<Home />} />
          <Route path="users" element={<Users />}>
            <Route path=":userId" element={<UserDetail />} />
          </Route>
          <Route path="*" element={<NotFound />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}

// 嵌套路由布局
function Layout() {
  return (
    <div>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/users">Users</Link>
      </nav>
      <Outlet /> {/* 子路由渲染位置 */}
    </div>
  );
}

// 获取参数
function UserDetail() {
  const { userId } = useParams();
  const [searchParams, setSearchParams] = useSearchParams();
  const navigate = useNavigate();
  
  return (
    <div>
      <h1>User: {userId}</h1>
      <button onClick={() => navigate(-1)}>返回</button>
    </div>
  );
}
```

---

### 7.2 路由守卫实现

```jsx
// 认证守卫组件
function RequireAuth({ children }) {
  const { user } = useAuth();
  const location = useLocation();
  
  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  return children;
}

// 使用
<Route
  path="/dashboard"
  element={
    <RequireAuth>
      <Dashboard />
    </RequireAuth>
  }
/>
```

---

## 八、实战与最佳实践

### 8.1 项目结构推荐

```
src/
├── components/         # 通用组件
│   ├── Button/
│   │   ├── index.tsx
│   │   ├── Button.module.css
│   │   └── Button.test.tsx
│   └── ...
├── pages/              # 页面组件
│   ├── Home/
│   └── Dashboard/
├── hooks/              # 自定义 Hooks
├── contexts/           # Context 定义
├── services/           # API 服务
├── utils/              # 工具函数
├── types/              # TypeScript 类型
├── constants/          # 常量定义
└── App.tsx
```

---

### 8.2 错误边界

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // 上报错误到监控服务
    logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-fallback">
          <h2>出错了！</h2>
          <button onClick={() => this.setState({ hasError: false })}>
            重试
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}

// 使用
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>
```

---

### 8.3 常见面试编程题

#### 题目1：实现 useDebounce Hook

```jsx
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}
```

#### 题目2：实现 usePrevious Hook

```jsx
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
}
```

#### 题目3：实现一个简单的 useState

```jsx
let state;
let stateIndex = 0;
const stateArray = [];

function useState(initialValue) {
  const currentIndex = stateIndex;
  
  if (stateArray[currentIndex] === undefined) {
    stateArray[currentIndex] = initialValue;
  }
  
  const setState = (newValue) => {
    stateArray[currentIndex] = typeof newValue === 'function'
      ? newValue(stateArray[currentIndex])
      : newValue;
    render(); // 触发重新渲染
  };
  
  stateIndex++;
  return [stateArray[currentIndex], setState];
}
```

---

## 快速复习清单 ✅

- [ ] React 核心理念：声明式、组件化、单向数据流
- [ ] JSX 本质是 `React.createElement` 的语法糖
- [ ] 函数组件 + Hooks 是推荐的写法
- [ ] useState 更新是异步批处理的
- [ ] useEffect 依赖数组的正确使用
- [ ] useMemo 缓存值，useCallback 缓存函数
- [ ] React.memo 优化函数组件渲染
- [ ] Context 适合低频更新的全局状态
- [ ] Fiber 实现了可中断的异步渲染
- [ ] 虚拟 DOM + Diff 算法提高渲染效率

---

## 相关资源 📚

- [React 官方文档](https://react.dev)
- [React GitHub](https://github.com/facebook/react)
- [React Patterns](https://reactpatterns.com)

---

> 💡 **提示**：面试时不仅要知道"是什么"，更要理解"为什么"和"怎么实现"。结合项目经验来回答问题会更有说服力。
