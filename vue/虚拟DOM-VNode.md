# 虚拟DOM - VNode

> 🔗 **关联笔记**：[[Vue3源码实战笔记]]
> 📅 **创建时间**：2024-12-24
> 📁 **源码位置**：`packages/runtime-core/src/vnode.ts`

---

## 📖 什么是 VNode

VNode（Virtual Node）是对真实 DOM 的 JavaScript 对象描述。

```javascript
// 真实 DOM
<div id="app" class="container">
  <span>Hello</span>
</div>

// 对应的 VNode
{
  type: 'div',
  props: { id: 'app', class: 'container' },
  children: [
    {
      type: 'span',
      props: null,
      children: 'Hello'
    }
  ]
}
```

---

## 🔧 VNode 结构

```typescript
interface VNode {
  type: VNodeTypes           // 节点类型
  props: VNodeProps | null   // 属性
  children: VNodeChildren    // 子节点
  key: string | number | null // diff 用的 key
  
  el: Element | null         // 对应的真实 DOM
  shapeFlag: number          // 节点类型标记
  patchFlag: number          // 优化标记
  
  component: ComponentInstance | null // 组件实例
  // ...
}
```

---

## 🏷️ ShapeFlag 类型标记

```typescript
export const enum ShapeFlags {
  ELEMENT = 1,                    // 普通元素
  FUNCTIONAL_COMPONENT = 1 << 1,  // 函数组件
  STATEFUL_COMPONENT = 1 << 2,    // 有状态组件
  TEXT_CHILDREN = 1 << 3,         // 文本子节点
  ARRAY_CHILDREN = 1 << 4,        // 数组子节点
  SLOTS_CHILDREN = 1 << 5,        // 插槽子节点
  // ...
}

// 使用位运算判断类型
if (shapeFlag & ShapeFlags.ELEMENT) {
  // 是普通元素
}
```

---

## 🎯 createVNode

```typescript
function createVNode(type, props, children) {
  const vnode = {
    type,
    props,
    children,
    key: props?.key ?? null,
    el: null,
    shapeFlag: getShapeFlag(type)
  }
  
  // 标准化 children
  if (typeof children === 'string') {
    vnode.shapeFlag |= ShapeFlags.TEXT_CHILDREN
  } else if (Array.isArray(children)) {
    vnode.shapeFlag |= ShapeFlags.ARRAY_CHILDREN
  }
  
  return vnode
}
```

---

## 📚 相关笔记

- [[Vue3源码实战笔记]] - 主笔记
- [[Diff算法详解]] - Diff 算法
- [[Renderer渲染器]] - 渲染器

#Vue3 #VNode #虚拟DOM #源码分析
