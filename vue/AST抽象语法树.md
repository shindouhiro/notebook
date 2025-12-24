# AST 抽象语法树

> 🔗 **关联笔记**：[[Vue3源码实战笔记]]
> 📅 **创建时间**：2024-12-24
> 📁 **源码位置**：`packages/compiler-core/src/ast.ts`

---

## 📖 AST 节点类型

```typescript
export const enum NodeTypes {
  ROOT,           // 根节点
  ELEMENT,        // 元素节点
  TEXT,           // 文本节点
  INTERPOLATION,  // 插值 {{ }}
  SIMPLE_EXPRESSION, // 简单表达式
  ATTRIBUTE,      // 属性
  DIRECTIVE,      // 指令
  // ...
}
```

---

## 🔧 节点结构

```typescript
// 元素节点
interface ElementNode {
  type: NodeTypes.ELEMENT
  tag: string
  props: Array<AttributeNode | DirectiveNode>
  children: TemplateChildNode[]
  isSelfClosing: boolean
}

// 插值节点
interface InterpolationNode {
  type: NodeTypes.INTERPOLATION
  content: ExpressionNode
}

// 文本节点
interface TextNode {
  type: NodeTypes.TEXT
  content: string
}
```

---

## 📊 示例

```html
<div id="app">{{ msg }}</div>
```

AST 结构：

```javascript
{
  type: NodeTypes.ROOT,
  children: [{
    type: NodeTypes.ELEMENT,
    tag: 'div',
    props: [{
      type: NodeTypes.ATTRIBUTE,
      name: 'id',
      value: { content: 'app' }
    }],
    children: [{
      type: NodeTypes.INTERPOLATION,
      content: {
        type: NodeTypes.SIMPLE_EXPRESSION,
        content: 'msg'
      }
    }]
  }]
}
```

---

## 📚 相关笔记

- [[Vue3源码实战笔记]] - 主笔记
- [[模板编译流程]] - 编译流程

#Vue3 #AST #编译器 #源码分析
