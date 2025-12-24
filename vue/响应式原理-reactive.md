# 响应式原理 - reactive

> 🔗 **关联笔记**：[[Vue3源码实战笔记]]
> 📅 **创建时间**：2024-12-24
> 📁 **源码位置**：`packages/reactivity/src/reactive.ts`

---

## 📖 概述

`reactive` 是 Vue 3 响应式系统的核心 API，用于创建深度响应式对象。它基于 ES6 的 **Proxy** 实现，相比 Vue 2 的 `Object.defineProperty` 有很大改进。

---

## 🆚 Vue 2 vs Vue 3

| 特性 | Vue 2 (defineProperty) | Vue 3 (Proxy) |
|------|------------------------|---------------|
| 监听新增属性 | ❌ 需要 `$set` | ✅ 自动监听 |
| 监听数组索引 | ❌ 需要特殊处理 | ✅ 自动监听 |
| 监听删除属性 | ❌ 需要 `$delete` | ✅ 自动监听 |
| 性能 | 初始化时递归遍历 | 惰性代理，按需转换 |

---

## 🔧 核心实现

### 1. reactive 函数入口

```typescript
// packages/reactivity/src/reactive.ts

export function reactive<T extends object>(target: T): UnwrapNestedRefs<T> {
  // 如果是只读代理，直接返回
  if (isReadonly(target)) {
    return target
  }
  
  return createReactiveObject(
    target,
    false,
    mutableHandlers,      // 普通对象的处理器
    mutableCollectionHandlers, // 集合类型的处理器
    reactiveMap           // 缓存 Map
  )
}
```

### 2. 创建响应式对象

```typescript
function createReactiveObject(
  target: Target,
  isReadonly: boolean,
  baseHandlers: ProxyHandler<any>,
  collectionHandlers: ProxyHandler<any>,
  proxyMap: WeakMap<Target, any>
) {
  // 只能代理对象类型
  if (!isObject(target)) {
    return target
  }
  
  // 已经是代理对象，直接返回
  if (target[ReactiveFlags.RAW] && !(isReadonly && target[ReactiveFlags.IS_REACTIVE])) {
    return target
  }
  
  // 查找缓存，避免重复代理
  const existingProxy = proxyMap.get(target)
  if (existingProxy) {
    return existingProxy
  }
  
  // 获取目标类型
  const targetType = getTargetType(target)
  if (targetType === TargetType.INVALID) {
    return target
  }
  
  // 创建 Proxy 代理
  const proxy = new Proxy(
    target,
    targetType === TargetType.COLLECTION ? collectionHandlers : baseHandlers
  )
  
  // 缓存代理对象
  proxyMap.set(target, proxy)
  
  return proxy
}
```

### 3. mutableHandlers (核心拦截器)

```typescript
// packages/reactivity/src/baseHandlers.ts

export const mutableHandlers: ProxyHandler<object> = {
  get,
  set,
  deleteProperty,
  has,
  ownKeys
}

// ========== GET 拦截 ==========
function get(target: Target, key: string | symbol, receiver: object) {
  // 处理特殊标记
  if (key === ReactiveFlags.IS_REACTIVE) {
    return true
  }
  if (key === ReactiveFlags.RAW) {
    return target
  }
  
  const res = Reflect.get(target, key, receiver)
  
  // 🔥 收集依赖 (核心!)
  track(target, TrackOpTypes.GET, key)
  
  // 如果是对象，递归代理 (惰性转换)
  if (isObject(res)) {
    return reactive(res)
  }
  
  return res
}

// ========== SET 拦截 ==========
function set(
  target: object,
  key: string | symbol,
  value: unknown,
  receiver: object
): boolean {
  const oldValue = (target as any)[key]
  
  // 判断是新增还是修改
  const hadKey = hasOwn(target, key)
  
  const result = Reflect.set(target, key, value, receiver)
  
  // 🔥 触发更新 (核心!)
  if (!hadKey) {
    // 新增属性
    trigger(target, TriggerOpTypes.ADD, key, value)
  } else if (hasChanged(value, oldValue)) {
    // 修改属性
    trigger(target, TriggerOpTypes.SET, key, value, oldValue)
  }
  
  return result
}

// ========== DELETE 拦截 ==========
function deleteProperty(target: object, key: string | symbol): boolean {
  const hadKey = hasOwn(target, key)
  const result = Reflect.deleteProperty(target, key)
  
  if (result && hadKey) {
    trigger(target, TriggerOpTypes.DELETE, key)
  }
  
  return result
}
```

---

## 🎯 track 依赖收集

```typescript
// packages/reactivity/src/effect.ts

// 全局变量：当前正在执行的 effect
let activeEffect: ReactiveEffect | undefined

// 依赖存储结构：target → key → effects
const targetMap = new WeakMap<any, KeyToDepMap>()

export function track(target: object, type: TrackOpTypes, key: unknown) {
  // 如果没有 activeEffect，不需要收集
  if (!activeEffect) return
  
  // 获取 target 对应的 depsMap
  let depsMap = targetMap.get(target)
  if (!depsMap) {
    targetMap.set(target, (depsMap = new Map()))
  }
  
  // 获取 key 对应的 dep (Set)
  let dep = depsMap.get(key)
  if (!dep) {
    depsMap.set(key, (dep = new Set()))
  }
  
  // 添加当前 effect 到依赖集合
  if (!dep.has(activeEffect)) {
    dep.add(activeEffect)
    activeEffect.deps.push(dep)
  }
}
```

### 依赖存储结构

```
targetMap (WeakMap)
  └── target (原始对象)
        └── depsMap (Map)
              ├── key1 → dep (Set) → [effect1, effect2]
              ├── key2 → dep (Set) → [effect3]
              └── key3 → dep (Set) → [effect1, effect4]
```

---

## ⚡ trigger 触发更新

```typescript
export function trigger(
  target: object,
  type: TriggerOpTypes,
  key?: unknown,
  newValue?: unknown,
  oldValue?: unknown
) {
  const depsMap = targetMap.get(target)
  if (!depsMap) return
  
  // 收集要执行的 effects
  const effects: ReactiveEffect[] = []
  
  const add = (effectsToAdd: Set<ReactiveEffect> | undefined) => {
    if (effectsToAdd) {
      effectsToAdd.forEach(effect => {
        if (effect !== activeEffect) {
          effects.push(effect)
        }
      })
    }
  }
  
  // 根据操作类型收集 effects
  if (type === TriggerOpTypes.CLEAR) {
    // 清空集合，触发所有依赖
    depsMap.forEach(add)
  } else {
    // 收集 key 对应的依赖
    if (key !== void 0) {
      add(depsMap.get(key))
    }
    
    // 新增/删除属性时，触发 length 或迭代器依赖
    if (type === TriggerOpTypes.ADD || type === TriggerOpTypes.DELETE) {
      add(depsMap.get(isArray(target) ? 'length' : ITERATE_KEY))
    }
  }
  
  // 执行所有 effects
  effects.forEach(effect => {
    if (effect.scheduler) {
      effect.scheduler()
    } else {
      effect.run()
    }
  })
}
```

---

## 🧪 手写简易 reactive

```javascript
// 简化版 reactive 实现

const targetMap = new WeakMap()
let activeEffect = null

// 创建响应式对象
function reactive(target) {
  return new Proxy(target, {
    get(target, key, receiver) {
      const result = Reflect.get(target, key, receiver)
      
      // 收集依赖
      track(target, key)
      
      // 深度代理
      if (typeof result === 'object' && result !== null) {
        return reactive(result)
      }
      
      return result
    },
    
    set(target, key, value, receiver) {
      const oldValue = target[key]
      const result = Reflect.set(target, key, value, receiver)
      
      // 触发更新
      if (oldValue !== value) {
        trigger(target, key)
      }
      
      return result
    }
  })
}

// 收集依赖
function track(target, key) {
  if (!activeEffect) return
  
  let depsMap = targetMap.get(target)
  if (!depsMap) {
    targetMap.set(target, (depsMap = new Map()))
  }
  
  let dep = depsMap.get(key)
  if (!dep) {
    depsMap.set(key, (dep = new Set()))
  }
  
  dep.add(activeEffect)
}

// 触发更新
function trigger(target, key) {
  const depsMap = targetMap.get(target)
  if (!depsMap) return
  
  const dep = depsMap.get(key)
  if (dep) {
    dep.forEach(effect => effect())
  }
}

// 副作用函数
function effect(fn) {
  activeEffect = fn
  fn()
  activeEffect = null
}

// ========== 测试 ==========
const state = reactive({ count: 0, name: 'Vue' })

effect(() => {
  console.log('count changed:', state.count)
})

state.count++ // 输出: count changed: 1
state.count++ // 输出: count changed: 2
```

---

## 💡 关键点总结

1. **Proxy 代理**：拦截 get/set/delete 等操作
2. **惰性代理**：嵌套对象只在访问时才转换
3. **依赖收集**：get 时 track，建立属性与 effect 的映射
4. **触发更新**：set 时 trigger，执行相关 effect
5. **缓存机制**：同一对象只创建一个代理

---

## 📚 相关笔记

- [[Vue3源码实战笔记]] - 主笔记
- [[响应式原理-effect]] - effect 副作用函数
- [[响应式原理-ref]] - ref 实现

---

#Vue3 #响应式 #Proxy #源码分析
