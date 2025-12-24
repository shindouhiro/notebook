# Props 和 Emit

> 🔗 **关联笔记**：[[Vue3源码实战笔记]]
> 📅 **创建时间**：2024-12-24
> 📁 **源码位置**：`packages/runtime-core/src/componentProps.ts`

---

## 📖 Props 初始化

```typescript
function initProps(instance, rawProps) {
  const props = {}
  const attrs = {}
  
  const options = instance.type.props || {}
  
  for (const key in rawProps) {
    if (key in options) {
      // 声明的 props
      props[key] = rawProps[key]
    } else {
      // 未声明的放入 attrs
      attrs[key] = rawProps[key]
    }
  }
  
  // props 是浅层响应式
  instance.props = shallowReactive(props)
  instance.attrs = attrs
}
```

---

## 📖 Emit 实现

```typescript
function emit(instance, event, ...args) {
  const props = instance.vnode.props || {}
  
  // 转换事件名：click → onClick
  const handlerName = `on${capitalize(event)}`
  
  const handler = props[handlerName]
  if (handler) {
    handler(...args)
  }
}

// 使用
// 父组件: <Child @click="handleClick" />
// 子组件: emit('click', data)
```

---

## 📚 相关笔记

- [[Vue3源码实战笔记]] - 主笔记
- [[组件实例化流程]] - 组件挂载

#Vue3 #Props #Emit #源码分析
