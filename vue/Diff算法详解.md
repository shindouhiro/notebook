# Diff 算法详解

> 🔗 **关联笔记**：[[Vue3源码实战笔记]]
> 📅 **创建时间**：2024-12-24
> 📁 **源码位置**：`packages/runtime-core/src/renderer.ts`

---

## 📖 概述

Vue 3 使用**快速 Diff 算法**，相比 Vue 2 的双端 Diff 更加高效。

---

## 🔧 核心步骤

### 1. 预处理：头部相同节点

```typescript
// 从头部开始比较
while (i <= e1 && i <= e2) {
  const n1 = c1[i]
  const n2 = c2[i]
  if (isSameVNodeType(n1, n2)) {
    patch(n1, n2, ...)
  } else {
    break
  }
  i++
}
```

### 2. 预处理：尾部相同节点

```typescript
// 从尾部开始比较
while (i <= e1 && i <= e2) {
  const n1 = c1[e1]
  const n2 = c2[e2]
  if (isSameVNodeType(n1, n2)) {
    patch(n1, n2, ...)
  } else {
    break
  }
  e1--
  e2--
}
```

### 3. 处理新增/删除

```typescript
// 旧的遍历完，新的还有 → 新增
if (i > e1 && i <= e2) {
  while (i <= e2) {
    patch(null, c2[i], ...)
    i++
  }
}

// 新的遍历完，旧的还有 → 删除
else if (i > e2 && i <= e1) {
  while (i <= e1) {
    unmount(c1[i])
    i++
  }
}
```

### 4. 乱序对比（最长递增子序列）

```typescript
// 建立新节点 key → index 映射
const keyToNewIndexMap = new Map()
for (let i = s2; i <= e2; i++) {
  keyToNewIndexMap.set(c2[i].key, i)
}

// 遍历旧节点，找出需要移动的
const newIndexToOldIndexMap = new Array(toBePatched).fill(0)

for (let i = s1; i <= e1; i++) {
  const newIndex = keyToNewIndexMap.get(c1[i].key)
  if (newIndex !== undefined) {
    newIndexToOldIndexMap[newIndex - s2] = i + 1
    patch(c1[i], c2[newIndex], ...)
  } else {
    unmount(c1[i])
  }
}

// 计算最长递增子序列，最小化移动
const increasingNewIndexSequence = getSequence(newIndexToOldIndexMap)
```

---

## 📊 示例

```
旧: A B C D E F G
新: A B F C D E G

1. 头部相同: A B
2. 尾部相同: G
3. 乱序部分: [C D E F] → [F C D E]
4. 最长递增子序列: [C D E] (不移动)
5. 只需移动 F
```

---

## 📚 相关笔记

- [[Vue3源码实战笔记]] - 主笔记
- [[虚拟DOM-VNode]] - VNode 结构

#Vue3 #Diff算法 #源码分析
