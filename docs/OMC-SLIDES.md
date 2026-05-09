---
marp: true
theme: default
paginate: true
size: 16:9
style: |
  section {
    font-family: 'PingFang SC', 'Microsoft YaHei', sans-serif;
  }
  h1 {
    color: #1a1a2e;
  }
  h2 {
    color: #16213e;
  }
  code {
    background: #f0f0f0;
    padding: 2px 6px;
    border-radius: 4px;
  }
  table {
    font-size: 0.85em;
  }
  .columns {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1em;
  }
---

# oh-my-claudecode (OMC)

## Claude Code 的多智能体编排系统

**一句话：让 Claude Code 从"一个人干活"变成"一个团队协作"**

---

# 它是什么？

| 没有 OMC | 有 OMC |
|----------|--------|
| Claude = 一个全栈工程师 | Claude = 技术总监 + 专业团队 |
| 一个 AI 做所有事 | 编排器指挥多个专业 AI |
| 做完一步就停 | 持续工作直到完成 |
| 不了解你的项目 | 自动学习项目环境 |

---

# 安装（30 秒）

```bash
# 第一步：安装插件
/plugin marketplace add https://github.com/Yeachan-Heo/oh-my-claudecode
/plugin install oh-my-claudecode

# 第二步：初始化
/omc-setup

# 验证
/omc-doctor
```

**安装后不需要额外启动任何进程，自动生效。**

---

# 安装后发生了什么？

```
安装插件
  → hooks.json 注册到 Claude Code
  → 每次使用 Claude Code 时自动激活
  → 无需手动启动
```

OMC 是 Claude Code 的**插件**，通过 Hook 机制嵌入生命周期。

---

# 效果一：无感增强（你什么都不做）

即使正常使用 Claude Code，OMC 在后台默默做这些：

| 能力 | 说明 |
|------|------|
| 项目记忆 | 自动学习技术栈、目录结构、代码风格 |
| 规则注入 | 读文件时自动注入 .rules 规则 |
| 错误恢复 | 上下文溢出、编辑失败自动修复 |
| 上下文保护 | 压缩前自动保存关键信息 |
| 权限管理 | 安全命令自动批准 |

---

# 效果二：魔法关键词触发超能力

| 你输入 | 效果 |
|--------|------|
| `autopilot: 做个登录页面` | 全自动：分析→规划→写代码→测试→验证 |
| `ralph: 修复所有 lint 错误` | 不停工作直到全部修完 |
| `ulw 重构 auth 模块` | 最大并行化，多个子智能体同时工作 |
| `/team 3:executor "加单元测试"` | 3 个 worker 并行写测试 |
| `plan API 设计` | 先讨论再动手 |
| `ultrathink` | 切换深度推理模式 |

---

# 效果三：编排器在管着 Claude

### 没有 OMC

```
你: "重构这个模块"
Claude: 直接开始改代码（一个人干所有事）
```

### 有 OMC

```
你: "重构这个模块"
Claude（编排器）:
  → 派 explore agent 了解代码结构
  → 派 planner agent 制定计划
  → 派 executor agent 执行修改
  → 派 verifier agent 验证结果
```

---

# 效果四：持久模式（不完成不停）

### 没有 OMC
```
Claude: "我修了 3 个错误，还有 12 个。要继续吗？"
（停下来等你回复）
```

### 有 OMC（ralph 模式）
```
Claude: 修了 3 个 → 继续 → 修了 6 个 → 继续 → 全部 15 个修完
"所有错误已修复，以下是变更摘要。"
（自动完成才停）
```

---

# 架构总览

```
┌─────────────────────────────────────┐
│       Claude Code CLI（宿主）         │
│       ↕ hooks.json 生命周期钩子       │
├─────────────────────────────────────┤
│       Bridge Layer (bridge/)         │
│       Shell → Node.js 桥接           │
├─────────────────────────────────────┤
│       Core Engine (src/)             │
│  hooks | agents | tools | mcp | team │
├─────────────────────────────────────┤
│       外部资源                        │
│  skills/*.md | agents/*.md | .omc/   │
└─────────────────────────────────────┘
```

---

# 核心机制：Hooks 系统

Claude Code 在不同时机触发钩子，OMC 注入自定义逻辑：

| 事件 | 时机 | OMC 做什么 |
|------|------|-----------|
| SessionStart | 会话开始 | 初始化状态、加载项目记忆 |
| UserPromptSubmit | 用户发消息 | 检测关键词、注入技能 |
| PreToolUse | 工具调用前 | 强制委派、权限检查 |
| PostToolUse | 工具调用后 | 验证结果、学习模式 |
| Stop | Claude 停止 | 持久模式续跑 |
| PreCompact | 压缩前 | 保存关键状态 |

---

# 核心机制：持久模式原理

```
1. 用户输入 "ralph: 修复所有错误"
2. keyword-detector 检测到 "ralph" → 激活持久模式
3. Claude 开始工作...
4. Claude 想停止 → 触发 Stop hook
5. persistent-mode 检查：任务完成了吗？
   ├── 未完成 → 注入 "The boulder never stops. Continue..."
   │            → Claude 继续工作
   └── 已完成 → 正常停止
6. 循环直到完成或用户 /cancel
```

---

# 核心机制：编排器委派

```
Claude 尝试直接修改 src/auth.ts
  → PreToolUse hook 拦截
  → 判断：不在允许路径（.omc/、.claude/ 等）
  → 阻止操作 + 提醒："请委派给 executor"
  → Claude 改为派出 executor subagent
```

**目的：** 强制专业分工，避免编排器亲自下场写代码。

---

# 19 个专业智能体

| 类别 | 智能体 | 模型 | 职责 |
|------|--------|------|------|
| 探索 | explore | haiku | 快速搜索 |
| 分析 | analyst | opus | 深度分析 |
| 规划 | planner | opus | 任务分解 |
| 架构 | architect | opus | 系统设计 |
| 执行 | executor | sonnet | 代码实现 |
| 调试 | debugger | sonnet | 根因分析 |
| 验证 | verifier | sonnet | 完成度验证 |
| 审查 | code-reviewer | opus | 代码审查 |
| 测试 | test-engineer | sonnet | 测试编写 |

---

# 模型路由策略

**核心思想：用最便宜的模型完成能胜任的工作**

```
haiku（便宜快速）  → 简单查找、文档撰写
sonnet（平衡）    → 大多数实现和验证
opus（最强最贵）  → 架构决策、深度分析
```

**效果：节省 30-50% 的 token 消耗**

---

# Team 模式：真正的并行

```
Leader（主 Claude 会话）
  ├── Worker 1（独立 Git worktree）
  ├── Worker 2（独立 Git worktree）
  └── Worker N（独立 Git worktree）
```

**执行流水线：**
```
team-plan → team-prd → team-exec → team-verify → team-fix（循环）
```

每个 Worker 有独立的代码副本，互不干扰，最后合并。

---

# Autopilot 全自动流程

```
autopilot: build a REST API for tasks
```

```
Step 1: 展开需求 → 生成详细规格
Step 2: 规划 → 生成执行计划
Step 3: 执行 → 派 executor 写代码
Step 4: QA → 派 test-engineer 写测试
Step 5: 验证 → 运行测试、检查构建
Step 6: 完成 → 输出总结报告
```

全程无需人工干预。

---

# MCP 工具：给 Claude 的额外能力

| 工具 | 能力 |
|------|------|
| LSP hover | 查看类型定义 |
| LSP goto definition | 跳转到定义 |
| LSP find references | 查找引用 |
| LSP diagnostics | 获取编译错误 |
| AST grep | 结构化代码搜索 |
| AST replace | 结构化代码替换 |
| Python REPL | 执行 Python 代码 |

---

# 状态管理

所有状态存储在 `.omc/` 目录下（本地文件，不上传）：

```
.omc/
├── state/              模式状态（ralph、ultrawork 等）
├── notepad.md          压缩安全的记忆
├── project-memory.json 项目环境学习结果
├── plans/              规划文件
├── research/           研究结果
└── logs/               执行日志
```

---

# 常见问题

**Q: 和 Claude Code 是什么关系？**
A: OMC 是插件，Claude Code 是宿主。通过 hooks 嵌入生命周期。

**Q: 会增加 token 消耗吗？**
A: Hook 处理不消耗 API token（本地执行）。多智能体模式会增加总量，但模型路由节省 30-50%。

**Q: 数据安全吗？**
A: 全部本地存储，不上传任何数据到外部服务。

**Q: 支持哪些 AI？**
A: 主要 Claude，Team 模式可调用 Codex（OpenAI）和 Gemini（Google）。

---

# 技术栈

| 技术 | 用途 |
|------|------|
| TypeScript | 核心逻辑 |
| esbuild | 打包为 CJS bridge |
| Vitest | 测试 |
| MCP SDK | 工具服务器 |
| better-sqlite3 | 本地存储 |
| ast-grep | 结构化代码操作 |
| Zod | 运行时类型验证 |

---

# 一句话总结

> **安装 OMC = 给 Claude Code 装了一个"项目经理大脑"**
>
> 它自动把大任务拆给专业角色，
> 自动续跑不停，
> 自动学习你的项目。
>
> 你可以完全无感使用，
> 也可以用关键词主动触发高级模式。

---

# 快速上手

```bash
# 安装
/plugin marketplace add https://github.com/Yeachan-Heo/oh-my-claudecode
/plugin install oh-my-claudecode
/omc-setup

# 试试这些
autopilot: create a simple todo API
ralph: fix all TypeScript errors
/team 2:executor "add input validation"
```

**开始享受多智能体协作吧！**
