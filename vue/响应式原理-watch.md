# 响应式原理 - watch

> 🔗 **关联笔记**：[[Vue3源码实战笔记]]
> 📅 **创建时间**：2024-12-24
> 📁 **源码位置**：`packages/runtime-core/src/apiWatch.ts`

---

## 📖 概述

`watch` 和 `watchEffect` 用于执行副作用：

| API | 特点 |
|-----|------|
| `watchEffect` | 立即执行，自动收集依赖 |
| `watch` | 显式指定依赖，可获取新旧值 |

---

## 🔧 watch 核心实现

```typescript
function doWatch(source, cb, { immediate, deep, flush }) {
  // 1. 标准化 source
  let getter
  if (isRef(source)) {
    getter = () => source.value
  } else if (isReactive(source)) {
    getter = () => source
    deep = true
  } else if (isFunction(source)) {
    getter = source
  }
  
  // 2. 深度监听
  if (deep) {
    const baseGetter = getter
    getter = () => traverse(baseGetter())
  }
  
  // 3. 调度任务
  let oldValue
  const job = () => {
    const newValue = effect.run()
    if (hasChanged(newValue, oldValue)) {
      cb(newValue, oldValue)
      oldValue = newValue
    }
  }
  
  // 4. 创建 effect
  const effect = new ReactiveEffect(getter, job)
  
  // 5. 初始执行
  if (immediate) {
    job()
  } else {
    oldValue = effect.run()
  }
  
  return () => effect.stop()
}
```

---

## ⏰ flush 时机

- `sync` - 同步执行
- `pre` - 组件更新前（默认）
- `post` - DOM 更新后

---

## 📚 相关笔记

- [[Vue3源码实战笔记]] - 主笔记
- [[响应式原理-effect]] - effect 实现

#Vue3 #watch #源码分析
