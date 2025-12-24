# Renderer 渲染器

> 🔗 **关联笔记**：[[Vue3源码实战笔记]]
> 📅 **创建时间**：2024-12-24
> 📁 **源码位置**：`packages/runtime-core/src/renderer.ts`

---

## 📖 概述

渲染器负责将 VNode 渲染到目标平台。Vue 3 的渲染器是**平台无关**的。

---

## 🔧 createRenderer

```typescript
function createRenderer(options) {
  const {
    insert,
    remove,
    createElement,
    createText,
    setText,
    setElementText,
    patchProp
  } = options
  
  function render(vnode, container) {
    if (vnode) {
      patch(container._vnode, vnode, container)
    } else if (container._vnode) {
      unmount(container._vnode)
    }
    container._vnode = vnode
  }
  
  function patch(n1, n2, container) {
    const { type, shapeFlag } = n2
    
    switch (type) {
      case Text:
        processText(n1, n2, container)
        break
      case Fragment:
        processFragment(n1, n2, container)
        break
      default:
        if (shapeFlag & ShapeFlags.ELEMENT) {
          processElement(n1, n2, container)
        } else if (shapeFlag & ShapeFlags.COMPONENT) {
          processComponent(n1, n2, container)
        }
    }
  }
  
  return { render }
}
```

---

## 🎯 processElement

```typescript
function processElement(n1, n2, container) {
  if (!n1) {
    // 挂载
    mountElement(n2, container)
  } else {
    // 更新
    patchElement(n1, n2)
  }
}

function mountElement(vnode, container) {
  const el = (vnode.el = createElement(vnode.type))
  
  // 处理属性
  if (vnode.props) {
    for (const key in vnode.props) {
      patchProp(el, key, null, vnode.props[key])
    }
  }
  
  // 处理子节点
  if (vnode.shapeFlag & ShapeFlags.TEXT_CHILDREN) {
    setElementText(el, vnode.children)
  } else if (vnode.shapeFlag & ShapeFlags.ARRAY_CHILDREN) {
    mountChildren(vnode.children, el)
  }
  
  insert(el, container)
}
```

---

## 📚 相关笔记

- [[Vue3源码实战笔记]] - 主笔记
- [[虚拟DOM-VNode]] - VNode 结构
- [[Diff算法详解]] - Diff 算法

#Vue3 #Renderer #渲染器 #源码分析
