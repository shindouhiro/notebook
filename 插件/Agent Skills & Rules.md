# 🤖 Agent Skills 生态介绍与实战指南

> [!abstract] 三大仓库概览
> Antfu 打造了一个完整的 Agent Skills（AI 智能体技能）生态，主要包含以下三个核心项目：
> 1. **[skills-cli](https://github.com/antfu/skills-cli)**：核心的命令行工具，用于安装和管理各种 AI Agent 的技能。
> 2. **[skills](https://github.com/antfu/skills)**：Antfu 个人维护的一站式精选技能集合，主要面向 Vue/Vite/Nuxt 等现代前端栈。
> 3. **[skills-npm](https://github.com/antfu/skills-npm)**：一种创新的技能分发机制，允许 npm 包自带对应的 Agent 技能。

---

## 1. 🛠️ `skills-cli`: Agent 技能的包管理器

`skills-cli` (`npx skills`) 是一切的基础，它是管理所有 AI 代码助手（如 Cursor、Windsurf、Claude Code、Cline 等）技能的统一命令行入口。

### 🌟 为什么需要它？
各大 AI 代码助手虽然都支持 “自定义指令” 或 “技能 (Skills)”，但规则各异、存储路径不同。`skills-cli` 帮你做到：**一键从 GitHub/GitLab 或本地安装大牛们写好的顶级提示词 (Prompts)**，并自动适配到当前使用的 AI 助手。

### 🚀 实战使用

**安装指定技能**：
```bash
# 从 Github 仓库安装具体的技能
npx skills add vercel-labs/agent-skills --skill frontend-design

# 把技能全局安装到特定的 AI 助手（比如 cursor 和 windsurf）
npx skills add vercel-labs/agent-skills -g -a cursor -a windsurf
```

**其他常用命令**：
```bash
npx skills list    # 查看已安装的技能
npx skills update  # 更新技能
npx skills remove  # 移除技能
```

---

## 2. 🧰 `skills`: 现代前端的“神级”提示词库

这个仓库是 Antfu 整理和收集的精选技能包。如果你在做 Vue/Nuxt/Vite 生态的开发，这是不可多得的神器。

里面包含三种技能：
1. **个人维护 (Opinionated)**：Antfu 自己的代码习惯和最佳实践。
2. **官方文档生成**：从 Vue、Nuxt、Pinia、Tailwind 等官方文档中直接生成的精准开发上下文。
3. **Vendored (第三方抓取)**：比如 Slidev、VueUse 等知名库自带的技能。

### 🚀 实战使用

在你的前端项目中执行以下命令，一键安装所有配套技能：

```bash
# 安装所有的 Antfu 精选技能到当前项目
npx skills add antfu/skills --skill='*'

# 如果你想系统全局生效
npx skills add antfu/skills --skill='*' -g
```
> [!success] 效果预测
> 安装后，你的 Cursor/Windsurf 就会自动知道怎么用最新的 Vue 3 组合式 API、VueUse，甚至能完全模仿 Antfu 的代码审美来帮你写代码！

---

## 3. 📦 `skills-npm`: 伴随 npm 包下发的 AI 技能

以前 AI 技能都是单独放在 Github 上，你需要手动到处找。`skills-npm` 提出了一种革命性的模式：**当你安装一个 npm 包时，自动顺便安装它的专属 AI 技能。**

### 🌟 痛点解决
解决了传统模式下：技能与 npm 包版本不匹配、需手动跨仓库找提示词、团队无法自动同步技能配置等问题。

### 🚀 实战使用

在你的项目开发中，只需执行：

1️⃣ **安装工具**：
```bash
npm i -D skills-npm
```

2️⃣ **配置 `package.json`**（利用生命周期钩子）：
```json
{
  "private": true,
  "scripts": {
    "prepare": "skills-npm"
  }
}
```

3️⃣ **自动生效**：
以后每当你或你的团队成员运行 `npm install` 时，`skills-npm` 就会自动扫描 `node_modules` 中哪些库自带了 AI 技能，并将它们符号链接（symlink）到当前项目的配置目录中。

> [!tip] 推荐配置
> 记得在 `.gitignore` 中忽略自动生成的 npm 技能文件：
> ```text
> skills/npm-*
> ```

---

## 🌐 `skills.sh`: AI 时代技能的“应用商店”

最后，这个生态还有两个不得不提的关键辅助工具：连接生态枢纽的 **skills.sh** 和帮你统一管理配置的 **ruler**。

### 🌟 这是一个什么平台？
它是专为 AI 代码代理（AI Agents）打造的公开技能目录中心（Leaderboard & Discovery 平台），类似于我们在手机应用商店寻找 App。任何人、任何团队开源的 Agent 技能，都会在这里被索引、展示和排名（Trending / Hot / All Time）。

### 📦 包含的内容（Leaderboard 示例）
当你在该网站上浏览时，你会看到各路顶尖团队贡献的精选神级技能模板。例如：
- `vercel-react-best-practices` (Vercel 出品的 React 最佳实践)
- `frontend-design` (Anthropic 出品的前端设计模式)
- `web-design-guidelines` (Vercel 出品的网页设计准则)
- 以及微软 Azure 的各类云环境自动配置指令等……

### 🚀 与 CLI 的完美结合
当你逛 **skills.sh** 看到心仪的技能时，直接复制页面上提供的对应命令行安装代码，然后在终端里结合上面讲到的 `npx skills add <所在仓库>` 就可以瞬间将大厂打磨过的完美提示词和 AI 开发实践下载到你的本地，为你的 AI 赋能！

---

## 📏 `ruler`: 集中管理和分发给多款 AI 助手

在使用前面提到的那些优秀的框架和资源池下载了技能之后，如果你本身是个“工具控”，在项目里混用了不同的 AI 助手（比如既用 Cursor、又用 Claude Code，还有 GitHub Copilot 和 Windsurf等），你一定会面临一个极大的痛点——**配置路径太多、太散乱**。为了解决这个问题，社区推出了 **[Ruler](https://github.com/intellectronica/ruler)**。

### 🌟 这是一个什么工具？
`ruler` 是一个跨 AI 工具的**统一指令和技能分发管理器**。它主张“单一数据源 (Single Source of Truth)”，让你只在一个地方维护 AI 提示词和配置，再由工具自动推送给各种代理。

### 📦 核心特性与解决痛点
- **统一指令 (Rules)**：只需要在项目根目录维护唯一的 `.ruler/AGENTS.md` 文件。
- **统一技能 (Skills)**：你可以把前面通过 `skills-cli` 弄好的技能统一放在 `.ruler/skills/` 文件夹下。
- **🚀 统一 MCP (Model Context Protocol) 插件**：**这是超级强大的一点！** 除了发提示词，`ruler` 更能在 `.ruler/ruler.toml` 里统一配置所有的 MCP Server（例如：搜索网页用的、看数据库用的等）。一次配置，自动全家桶通用，极大地降低配置不同 AI 连通外部世界的成本。
- **支持复杂的嵌套规则结构**：非常适合 Monorepo。前后端不同的子目录可以拥有截然不同的 `.ruler/` 规则，工具能完美合并覆盖。
- **抹平平台差异**：执行 `ruler apply`，该工具会自动给你的不同编辑器生成专有“软配置”（如 `.cursor/skills/`、`.claude/skills/` 或者各自独立的 mcp.json），并帮你自动修改 `.gitignore` 隐藏这些碎文件，保证版本仓库干干净净。

### 🚀 实战使用
**1. 初始化项目全局配置**：
```bash
# 全局安装 ruler
npm install -g @intellectronica/ruler

# 到你的项目根目录执行初始化
ruler init
```
这会生成一个 `.ruler` 文件夹。

**2. 组装你的技能与指令**：
将通过 `npx skills` 下载的内容，或自己写的提示词丢到 `.ruler/` 内。

**3. 一键分发统治所有 AI 工具**：
```bash
ruler apply
```
瞬间，你的 Cursor、Claude Code 等助手就吃上了同一套最新鲜的“技能包”，不论你今天想用哪个工具开发，AI 小助手的“编程审美”都能始终保持同步的默契！

