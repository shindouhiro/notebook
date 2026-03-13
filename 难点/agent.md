# Agent 对话超长文本 Markdown 流式渲染性能优化

## 📌 业务场景与性能灾难
在调用 Agent/LLM 的对话接口（通常借由 Server-Sent Events (SSE) 实时流式返回数据）时，我们常规接入的 Markdown 解析器（如 `marked`、`markdown-it` 或 `react-markdown`）通常是全量解析的：
> 如果一段总对话内容非常漫长（哪怕积累了 2000 个字），此时网络数据流**每推进新增甚至 1 个字符，前端都要重新将整段极长的上下文灌入解析器重新编译一整棵巨大的 DOM 树。**

这带来的恶劣后果是计算与绘制成时间复杂度 $O(N^2)$ 的几何坍塌：
- 主线程 CPU 占用率持续高频飙升。
- 电脑风扇开始狂转。
- 浏览器输入框在打字时产生严重的粘滞感与掉帧卡顿。

## 🚀 核心落地思路

针对上述深水区问题，业内的标准工业级破局解法包含两套核心防御机制：
1. **分段解析机制与数据缓存（Memoization）**：基于换行或特定规则，拦截大模型反馈来的上下文文本，直接把它碎切成多个短小的 Block 段落（例如遇到空行且不在代码块中时，切割一次）。这样我们就能依靠闭包，把早已生成完毕的“老段落落”锁入缓存态，**将 CPU 压力限制在永远只解析打字机末尾那正在生成的几十个字**。
2. **底层借力框架虚拟 DOM（Virtual DOM Diffing）**：依托 React 或是 Vue 的同层 Patch 比对算力（或是基于 `React.memo` / `shouldComponentUpdate` 的显式屏蔽），一旦老段落字符串 `===` 之前的字符串内存，直接原地跳过比对切断整条更新链路，实现 60fps 丝滑渲染。

---

## 💻 工业级 Demo 实战演示 (以 React 为例)

### 1. 核心安全拆分算法 `markdownSplitter.ts`
做长段落拆解最大的难点在于遇到包裹大量换行和空行的**代码块**（` ``` ` 内部区域）。我们若简单粗暴地用两根换行符 `split('\n\n')` 去劈，那么含有空行的大片代码区域将被瞬间腰斩引发解析崩盘。必须借用极简状态机来规避核心沙盒。

```typescript
// utils/markdownSplitter.ts

export function splitMarkdownIntoBlocks(fullText: string): string[] {
  const blocks: string[] = [];
  const lines = fullText.split('\n');
  
  let currentBlockBuffer = '';
  let isInCodeBlock = false;

  for (let i = 0; i < lines.length; i++) {
    const line = lines[i];

    // 探测 Markdown 代码块沙盒开启/闭合状态
    if (line.trim().startsWith('```')) {
      isInCodeBlock = !isInCodeBlock;
    }

    currentBlockBuffer += line + '\n';

    // 核心安全拆分条件：非闭合沙盒内，且正好遇到了段落级空行
    if (!isInCodeBlock && line.trim() === '') {
      // 收割当前区块的内容（如前三个自然段）
      if (currentBlockBuffer.trim() !== '') {
        blocks.push(currentBlockBuffer);
      }
      currentBlockBuffer = ''; // 强效清空缓存盘，备载下一段
    }
  }

  // 兜尾机制：把最末尾那截没有空行结尾、正在生成的打字机片段塞入队列作为实时尾巴
  if (currentBlockBuffer) {
    blocks.push(currentBlockBuffer);
  }

  return blocks;
}
```

### 2. 局部锁死的隔热组件 `MarkdownBlock.tsx`
把 `marked.parse(局部短文)` 和针对 XSS 清洗的高额耗时计算包裹进该沙箱内。前线长得再猛，我岿然不动。

```tsx
// components/MarkdownBlock.tsx
import React, { useMemo, memo } from 'react';
import { marked } from 'marked';
import DOMPurify from 'dompurify'; // 防备 AI 恶念 HTML 吐出的必需品

interface Props {
  content: string;
}

// React.memo 充当断头台：只要 content 浅比较相等，绝不触碰内部重新解析
const MarkdownBlock: React.FC<Props> = memo(({ content }) => {
  
  // 以空间置换掉主耗时点
  const rawHtml = useMemo(() => {
    const html = marked.parse(content) as string;
    return DOMPurify.sanitize(html); 
  }, [content]);

  // Virtual Dom 只比对自己这几行的小模块 DOM Tree
  return (
    <div 
      className="md-block-segment" 
      dangerouslySetInnerHTML={{ __html: rawHtml }} 
    />
  );
  
}, (prev, next) => {
  return prev.content === next.content;
});

export default MarkdownBlock;
```

### 3. 流式 SSE 父级容器 `ChatStreamMessage.tsx`
整合并串联组装这台打字机流式渲染引擎：即使外层每次塞进来的 `fullStreamText` 是逐渐增加的巨无霸字符串巨兽，组件内通过一次低功耗的 $O(N)$ 循环打散成块之后，就能精细化下发指令。

```tsx
// components/ChatStreamMessage.tsx
import React, { useMemo } from 'react';
import { splitMarkdownIntoBlocks } from '../utils/markdownSplitter';
import MarkdownBlock from './MarkdownBlock';

interface Props {
  fullStreamText: string; 
}

const ChatStreamMessage: React.FC<Props> = ({ fullStreamText }) => {
  
  // 将整串字符串肢解为小块缓存数组
  const streamBlocks = useMemo(() => {
    return splitMarkdownIntoBlocks(fullStreamText);
  }, [fullStreamText]);

  return (
    <div className="message-container">
      {streamBlocks.map((blockContent, index) => {
        // 由于大模型的流数据本质是在末尾延伸，这代表：
        // 前方 length - 1 的元素由于字符串没变（在分段器里得到了保留），传到下面就被 memo 掐断了
        // 此刻真正在重新渲染的，永远并且只有本数组最后那 1 项内容！
        return (
          <MarkdownBlock 
            key={index} // 流式末尾安全追加时可以直接采用 index 作为粗略但稳定的句柄
            content={blockContent} 
          />
        );
      })}
    </div>
  );
};

export default ChatStreamMessage;
```

---
### 💡 给资深开发者的后续补充：
1. **行内闪烁打字机光标**：流式最后一段天然带着打字特征，只需通过对比 `index === streamBlocks.length - 1` 为末尾那一段送入一个特殊的 `boolean` `prop`，用来在其内部渲染最终的一枚代表生命力的 CSS 跳转光标 `|` 即可。
2. **相邻样式破损**：由于硬切断包裹成一个个独立的 `<div className="md-block-segment">`，有可能会出现本该紧凑的两段字突然产生了极宽的间隙（受外层基础样式影响）。可在 CSS 加入 `.md-block-segment + .md-block-segment { margin-top: -XXpx }` 修复相邻边缘视觉空差。
