# oh-my-claudecode (OMC) 项目讲解指南

> 本文档面向需要理解、使用或向他人介绍 OMC 的开发者。

---

## 一、项目定位

**oh-my-claudecode** 是 Claude Code CLI 的多智能体编排插件。

一句话总结：它让 Claude Code 从"一个 AI 做所有事"变成"一个编排器指挥多个专业 AI 协作完成任务"。

类比：
- Claude Code = 一个全栈工程师
- Claude Code + OMC = 一个技术总监 + 一支专业团队

---

## 二、安装与运行

### 2.1 安装方式

**方式一：Claude Code 插件市场（推荐）**

```bash
# 在 Claude Code 中执行
/plugin marketplace add https://github.com/Yeachan-Heo/oh-my-claudecode
/plugin install oh-my-claudecode
```

**方式二：npm 全局安装**

```bash
npm install -g oh-my-claude-sisyphus
```

**方式三：本地开发**

```bash
git clone https://github.com/Yeachan-Heo/oh-my-claudecode.git
cd oh-my-claudecode
npm install
npm run build
```

### 2.2 初始化配置

```bash
# 在 Claude Code 会话中执行
/omc-setup
```

这会：
1. 在 `~/.claude/` 下创建必要的目录结构
2. 注册 hooks 到 Claude Code 的 settings.json
3. 配置 MCP server
4. 验证环境依赖

### 2.3 验证安装

```bash
# 诊断工具
/omc-doctor
```

### 2.4 开发模式

```bash
# TypeScript 编译监听
npm run dev

# 全量监听（包括所有 bridge 构建）
npm run dev:full

# 运行测试
npm test
```

---

## 2.5 安装后到底有什么效果？

安装后**不需要额外启动任何进程**。OMC 是被动激活的：

```
安装插件 → hooks.json 注册到 Claude Code → 每次你用 Claude Code 时自动生效
```

### 效果一：无感增强（你什么都不做也有）

即使你正常使用 Claude Code，OMC 在后台默默做这些事：

| 能力 | 说明 |
|------|------|
| 项目记忆 | 自动学习你项目的技术栈、目录结构、代码风格 |
| 规则注入 | 读取文件时自动注入对应目录的 .rules 文件 |
| 错误恢复 | 遇到上下文溢出、编辑失败等错误自动修复 |
| 上下文保护 | 压缩前自动保存关键信息到 notepad |
| 权限管理 | 安全命令自动批准，减少弹窗 |
| 思考模式 | 输入 `ultrathink` 自动切换深度推理 |

### 效果二：用魔法关键词触发超能力

| 你输入 | 效果 |
|--------|------|
| `autopilot: 做个登录页面` | Claude 全自动完成：分析→规划→写代码→写测试→验证 |
| `ralph: 修复所有 lint 错误` | Claude 不停工作直到全部修完，不会中途停下 |
| `ulw 重构 auth 模块` | 最大并行化，多个子智能体同时工作 |
| `/team 3:executor "加单元测试"` | 3 个 worker 并行写测试 |
| `plan API 设计` | 进入规划模式，先讨论再动手 |

### 效果三：编排器在管着 Claude

**没有 OMC 时：**
```
你: "重构这个模块"
Claude: 直接开始改代码（一个人干所有事）
```

**有 OMC 时：**
```
你: "重构这个模块"
Claude（编排器）:
  → 派 explore agent 先了解代码结构
  → 派 planner agent 制定计划
  → 派 executor agent 执行修改
  → 派 verifier agent 验证结果
```

### 效果四：持久模式的体感

**没有 OMC：**
```
Claude: "我已经修复了 3 个错误，还有 12 个。你要我继续吗？"
（停下来等你回复）
```

**有 OMC（ralph 模式）：**
```
Claude: 修了 3 个 → 继续 → 修了 6 个 → 继续 → ... → 全部 15 个修完
"所有错误已修复，以下是变更摘要。"
（自动完成才停）
```

### 一句话总结

> 安装 OMC = 给 Claude Code 装了一个"项目经理大脑"。它自动把大任务拆给专业角色，自动续跑不停，自动学习你的项目。你可以完全无感使用，也可以用关键词主动触发高级模式。

---

## 三、架构总览

```
┌──────────────────────────────────────────────────────────┐
│                    Claude Code CLI（宿主）                  │
│                                                          │
│  hooks.json 注册生命周期钩子 ←→ Claude Code Hook System   │
└────────────────────────┬─────────────────────────────────┘
                         │ stdin/stdout JSON
┌────────────────────────▼─────────────────────────────────┐
│              Bridge Layer (bridge/)                        │
│  cli.cjs | mcp-server.cjs | team-bridge.cjs              │
│  Shell Script → Node.js CJS 入口 → TypeScript 逻辑       │
└────────────────────────┬─────────────────────────────────┘
                         │
┌────────────────────────▼─────────────────────────────────┐
│                   Core Engine (src/)                       │
│                                                          │
│  ┌─────────┐ ┌─────────┐ ┌───────┐ ┌───────┐           │
│  │  hooks  │ │ agents  │ │ tools │ │  mcp  │           │
│  └────┬────┘ └────┬────┘ └───┬───┘ └───┬───┘           │
│       │            │          │         │                │
│  ┌────▼────┐ ┌────▼────┐ ┌───▼───┐ ┌───▼───┐           │
│  │features │ │  team   │ │config │ │  lib  │           │
│  └─────────┘ └─────────┘ └───────┘ └───────┘           │
└──────────────────────────────────────────────────────────┘
                         │
┌────────────────────────▼─────────────────────────────────┐
│              外部资源                                      │
│  skills/*.md  |  agents/*.md  |  templates/  |  .omc/    │
└──────────────────────────────────────────────────────────┘
```

---

## 四、核心模块详解

### 4.1 Hooks 系统 — OMC 的神经网络

**原理：** Claude Code 提供了生命周期钩子机制。OMC 通过 `hooks/hooks.json` 注册 shell 脚本，在每个生命周期事件触发时执行自定义逻辑。

**数据流：**
```
Claude Code 事件 → shell 脚本 → scripts/run.cjs → TypeScript 处理 → JSON 响应 → Claude Code
```

**关键钩子一览：**

| 事件 | 文件 | 作用 |
|------|------|------|
| SessionStart | session-start.mjs | 初始化 OMC 状态、加载项目记忆 |
| UserPromptSubmit | keyword-detector.mjs | 检测魔法关键词（ralph、autopilot 等） |
| UserPromptSubmit | skill-injector.mjs | 注入匹配的 skill prompt |
| PreToolUse | pre-tool-enforcer.mjs | 编排器强制委派、阻止直接写操作 |
| PostToolUse | post-tool-verifier.mjs | 验证工具输出、学习项目模式 |
| PostToolUse | post-tool-rules-injector.mjs | 按路径注入 .rules 文件 |
| PermissionRequest | permission-handler.mjs | 自动批准安全命令 |
| SubagentStart/Stop | subagent-tracker.mjs | 跟踪子智能体生命周期 |
| PreCompact | pre-compact.mjs | 上下文压缩前保存关键状态 |
| Stop | persistent-mode.mjs | 持久模式续跑（核心机制） |
| Stop | code-simplifier.mjs | 完成后触发代码精简检查 |
| SessionEnd | session-end.mjs | 记录指标、清理临时状态 |

**Hook 响应格式：**
```json
{
  "continue": true,
  "stopMessage": "注入到 Claude 上下文的文本",
  "suppressOutput": true
}
```

- `continue: false` → 阻止当前操作
- `stopMessage` → 在 Stop 事件中注入续跑指令
- `suppressOutput` → 不向用户显示 hook 输出

### 4.2 智能体系统 — 专业分工

每个智能体是一个 `.md` 文件（`agents/*.md`），包含：
- frontmatter：name、description、model、level
- prompt：详细的角色定义、成功标准、约束

**智能体分类：**

| 类别 | 智能体 | 模型 | 职责 |
|------|--------|------|------|
| 探索 | explore | haiku | 快速代码搜索、文件定位 |
| 分析 | analyst | opus | 深度代码分析、架构评估 |
| 规划 | planner | opus | 任务分解、执行计划制定 |
| 架构 | architect | opus | 系统设计、技术决策 |
| 执行 | executor | sonnet | 代码实现（核心 worker） |
| 调试 | debugger | sonnet | 根因分析、故障诊断 |
| 验证 | verifier | sonnet | 完成度验证、证据收集 |
| 审查 | code-reviewer | opus | 代码质量审查 |
| 安全 | security-reviewer | sonnet | 安全漏洞检测 |
| 测试 | test-engineer | sonnet | 测试编写与执行 |
| 设计 | designer | sonnet | 前端/UI 设计 |
| 文档 | writer | haiku | 文档撰写 |
| 研究 | document-specialist | sonnet | 外部文档查阅 |
| 追踪 | tracer | sonnet | 证据链追踪 |
| 精简 | code-simplifier | opus | 代码复杂度降低 |
| 批评 | critic | opus | 方案批判性审查 |
| 科学 | scientist | sonnet | 假设验证实验 |
| QA | qa-tester | sonnet | 端到端质量保证 |
| Git | git-master | sonnet | Git 操作专家 |

**模型路由策略：**
- haiku（便宜快速）→ 简单查找、文档撰写
- sonnet（平衡）→ 大多数实现和验证工作
- opus（最强）→ 架构决策、深度分析、代码审查

### 4.3 Skills 系统 — 可调用的工作流

Skills 是预定义的复杂工作流，用户通过 `/oh-my-claudecode:<name>` 或魔法关键词触发。

**核心 Skills：**

| Skill | 触发方式 | 功能 |
|-------|---------|------|
| autopilot | `autopilot: <task>` | 全自动：展开→规划→执行→QA→验证 |
| ralph | `ralph: <task>` | 持久循环，不完成不停止 |
| ultrawork | `ulw <task>` | 最大并行化深度工作 |
| team | `/team N:executor "task"` | 多 worker 并行协作 |
| ccg | `/ccg <task>` | Claude + Codex + Gemini 三模型协作 |
| deep-interview | `/deep-interview "idea"` | 苏格拉底式需求澄清 |
| trace | `/trace <issue>` | 证据驱动的问题追踪 |
| ralplan | `ralplan <feature>` | 迭代规划直到共识 |
| omc-setup | `/omc-setup` | 环境初始化 |
| omc-doctor | `/omc-doctor` | 诊断修复 |

**Skill 文件结构（`skills/<name>/SKILL.md`）：**
```markdown
---
name: autopilot
description: Autonomous end-to-end execution
triggers: ["autopilot"]
---

<skill prompt content>
```

### 4.4 Team 系统 — 真正的并行执行

Team 是 OMC 最复杂的子系统，实现了多个 Claude 实例并行工作。

**架构：**
```
Leader (主 Claude 会话)
  ├── Worker 1 (tmux pane / Claude Agent)
  ├── Worker 2
  └── Worker N
```

**执行流水线：**
```
team-plan → team-prd → team-exec → team-verify → team-fix (循环)
```

**关键组件：**
- `dispatch-queue.ts` — 任务分发队列
- `message-router.ts` — Leader ↔ Worker 消息路由
- `heartbeat.ts` — Worker 健康检查
- `git-worktree.ts` — 每个 Worker 独立的 Git worktree
- `merge-orchestrator.ts` — 多 Worker 产出合并
- `tmux-session.ts` — tmux 会话管理
- `phase-controller.ts` — 阶段流转控制
- `governance.ts` — 治理规则（防止 Worker 越权）

### 4.5 MCP Server — 暴露给 Claude 的工具

OMC 通过 MCP（Model Context Protocol）向 Claude Code 提供额外工具：

| 工具类别 | 功能 |
|---------|------|
| LSP 工具 | hover、goto definition、find references、diagnostics |
| AST 工具 | ast-grep 搜索和替换（结构化代码操作） |
| Python REPL | 内嵌 Python 执行环境 |
| State 工具 | 读写 OMC 内部状态 |
| Notepad 工具 | 压缩安全的分层记忆 |
| Memory 工具 | 项目环境自动学习 |
| Trace 工具 | 流程追踪记录 |

### 4.6 Features 模块 — 增强能力

| 模块 | 作用 |
|------|------|
| magic-keywords | 检测并处理魔法关键词 |
| background-tasks | 后台任务管理 |
| continuation-enforcement | 防止 Claude 提前停止 |
| boulder-state | "巨石"状态（持久任务追踪） |
| context-injector | 上下文自动注入 |
| auto-update | 自动检查更新 |
| delegation-routing | 智能委派路由 |
| model-routing | 模型选择策略 |
| session-history-search | 跨会话历史搜索 |

---

## 五、核心机制详解

### 5.1 持久模式（The Boulder Never Stops）

这是 OMC 最核心的创新。普通 Claude Code 在完成一轮回复后会停止，但 OMC 的持久模式让它持续工作直到任务真正完成。

**原理：**
```
1. 用户输入 "ralph: refactor auth module"
2. keyword-detector 检测到 "ralph"，激活持久模式
3. Claude 开始工作...
4. Claude 认为可以停止了 → 触发 Stop hook
5. persistent-mode.mjs 检查：ralph 状态是否标记完成？
   - 未完成 → 返回 stopMessage: "The boulder never stops. Continue..."
   - Claude Code 将 stopMessage 作为新输入注入
   - Claude 继续工作
6. 循环直到任务完成或用户执行 /cancel
```

**状态文件：** `.omc/state/ralph.json`
```json
{
  "active": true,
  "iteration": 3,
  "startedAt": "2025-01-01T00:00:00Z",
  "task": "refactor auth module"
}
```

### 5.2 编排器委派强制（Orchestrator Enforcement）

防止主 Claude 直接写代码，强制它委派给专业智能体。

**原理：**
```
1. Claude 尝试调用 Edit 工具修改 src/auth.ts
2. PreToolUse hook 触发 → omc-orchestrator 检查
3. 判断：src/auth.ts 不在允许路径（.omc/、.claude/ 等）
4. 返回 continue: false + 提醒消息："请委派给 executor"
5. Claude 改为派出 executor subagent 来执行修改
```

**允许直接操作的路径：** `.omc/`、`.claude/`、`CLAUDE.md`、`AGENTS.md`

### 5.3 关键词检测与技能注入

**流程：**
```
用户输入 "autopilot: build a todo app"
  │
  ▼ keyword-detector
  ├── 去除代码块
  ├── 匹配关键词表
  ├── 发现 "autopilot"
  └── 返回 [MAGIC KEYWORD: autopilot]
  │
  ▼ skill-injector
  ├── 查找 skills/autopilot/SKILL.md
  ├── 读取完整 skill prompt
  └── 注入到 Claude 上下文
  │
  ▼ Claude 按照 skill prompt 执行完整工作流
```

### 5.4 Notepad — 压缩安全的记忆

Claude Code 会在上下文窗口接近极限时压缩历史消息。Notepad 确保关键信息在压缩后仍然可用。

**三层结构：**
```markdown
## Priority Context（永不丢失）
当前任务目标、关键约束

## Working Memory（按需保留）
中间发现、临时状态

## Manual（用户手动添加）
用户显式要求记住的内容
```

**触发时机：** PreCompact hook 在压缩前将关键状态导出到 notepad。

### 5.5 项目记忆（Project Memory）

自动学习项目环境，无需用户配置。

**学习内容：**
- 技术栈（语言、框架、版本）
- 构建工具和命令
- 代码规范（命名、导入风格）
- 目录结构
- 热点路径（频繁访问的文件）

**学习时机：** PostToolUse hook 分析工具输出，提取项目信息。

---

## 六、数据流完整示例

### 场景：用户输入 `autopilot: build a REST API for tasks`

```
Step 1: UserPromptSubmit
  ├── keyword-detector → 检测到 "autopilot"
  └── skill-injector → 注入 autopilot skill prompt

Step 2: Autopilot 展开阶段（Expansion）
  ├── Claude 分析需求
  ├── 派出 explore agent 了解现有代码
  └── 生成详细需求规格 → .omc/autopilot/spec.md

Step 3: Autopilot 规划阶段（Planning）
  ├── 派出 planner agent
  └── 生成执行计划 → .omc/autopilot/plan.md

Step 4: Autopilot 执行阶段（Execution）
  ├── 派出 executor agent(s)
  ├── PreToolUse 确保只有 executor 在写代码
  ├── PostToolUse 验证每步结果
  └── 代码实现完成

Step 5: Autopilot QA 阶段
  ├── 派出 test-engineer 编写测试
  ├── 派出 verifier 验证完成度
  └── 如果失败 → 回到 Step 4 修复

Step 6: Autopilot 验证阶段（Validation）
  ├── 运行完整测试套件
  ├── 检查构建是否通过
  └── 生成总结报告

Step 7: Stop hook
  ├── autopilot 状态标记完成
  └── 正常停止（不续跑）
```

---

## 七、配置体系

### 7.1 配置文件位置

| 文件 | 作用 |
|------|------|
| `.claude/omc.jsonc` | 项目级 OMC 配置 |
| `~/.claude/omc_config.json` | 用户级全局配置 |
| `hooks/hooks.json` | Hook 注册表 |
| `.mcp.json` | MCP server 配置 |
| `.claude-plugin/plugin.json` | 插件元数据 |

### 7.2 关键配置项

```jsonc
// .claude/omc.jsonc
{
  "features": {
    "continuationEnforcement": true,  // 续跑强制
    "autoContextInjection": true,     // 自动上下文注入
    "lspTools": true,                 // LSP 工具
    "astTools": true                  // AST 工具
  },
  "permissions": {
    "allowBash": true,
    "allowEdit": true,
    "allowWrite": true
  },
  "team": {
    "roleRouting": {
      "reviewer": "codex",
      "critic": "gemini"
    }
  }
}
```

---

## 八、目录结构速查

```
oh-my-claudecode/
├── agents/              # 智能体 prompt 定义（.md 文件）
├── bridge/              # Shell → Node.js 桥接入口（CJS）
├── hooks/               # hooks.json（Claude Code hook 注册）
├── skills/              # 用户可调用的工作流技能
├── src/
│   ├── hooks/           # Hook 处理逻辑（最大模块）
│   │   ├── keyword-detector/    # 关键词检测
│   │   ├── ralph/               # 持久模式（循环+PRD+进度+验证）
│   │   ├── autopilot/           # 全自动执行
│   │   ├── ultrawork/           # 深度工作模式
│   │   ├── omc-orchestrator/    # 编排器委派强制
│   │   ├── recovery/            # 错误恢复
│   │   ├── notepad/             # 压缩安全记忆
│   │   ├── project-memory/      # 项目环境学习
│   │   ├── rules-injector/      # 规则文件注入
│   │   ├── think-mode/          # 深度思考模式
│   │   ├── learner/             # 技能自动学习
│   │   └── ...
│   ├── agents/          # 智能体 TypeScript 定义与工厂
│   ├── team/            # 多 Worker 并行系统
│   ├── mcp/             # MCP server 实现
│   ├── tools/           # LSP/AST/REPL 工具
│   ├── features/        # 功能模块
│   ├── config/          # 配置加载
│   ├── lib/             # 共享库
│   └── index.ts         # 主入口（createOmcSession）
├── scripts/             # 构建脚本 + hook 入口脚本
├── templates/           # 项目模板
├── tests/               # 测试
├── docs/                # 文档
└── .omc/                # 运行时状态（gitignore）
```

---

## 九、常用命令速查

### 工作流命令

| 命令 | 说明 |
|------|------|
| `autopilot: <task>` | 全自动执行任务 |
| `ralph: <task>` | 持久模式，不完成不停 |
| `ulw <task>` | 最大并行深度工作 |
| `/team N:executor "task"` | N 个 worker 并行执行 |
| `/ccg <task>` | Claude+Codex+Gemini 协作 |
| `/deep-interview "idea"` | 需求澄清访谈 |
| `plan <feature>` | 规划模式 |
| `/cancel` | 取消当前执行模式 |

### 管理命令

| 命令 | 说明 |
|------|------|
| `/omc-setup` | 初始化/重新配置 |
| `/omc-doctor` | 诊断问题 |
| `/omc-help` | 查看帮助 |
| `/hud` | 查看 HUD 状态 |
| `/skill list` | 列出已学习的技能 |
| `omc wait --start` | 启动速率限制自动恢复 |

---

## 十、讲解要点（给演示者）

### 核心卖点（3 分钟版）

1. **零配置增强** — 安装即用，不改变 Claude Code 使用习惯
2. **专业分工** — 19 个智能体各司其职，比单一 AI 更精准
3. **永不放弃** — 持久模式确保复杂任务完整完成
4. **成本优化** — 简单任务用便宜模型，复杂任务才用贵的
5. **真正并行** — Team 模式让多个 AI 同时工作

### 演示建议

1. **入门演示：** `autopilot: create a simple Express API with CRUD for todos`
   - 展示全自动流程：需求展开 → 规划 → 实现 → 测试 → 验证

2. **持久模式演示：** `ralph: fix all TypeScript errors in this project`
   - 展示 Claude 不会中途停止，持续修复直到零错误

3. **并行演示：** `/team 3:executor "add input validation to all API endpoints"`
   - 展示多个 worker 同时工作，最后合并

### 常见问题

**Q: OMC 和 Claude Code 是什么关系？**
A: OMC 是 Claude Code 的插件。Claude Code 是宿主，OMC 通过 hooks 机制嵌入其生命周期，增强其能力。

**Q: 会增加多少 token 消耗？**
A: Hook 处理本身不消耗 API token（本地 Node.js 执行）。但多智能体模式会因为派出子智能体而增加总 token 用量。模型路由策略（简单任务用 haiku）可以节省 30-50%。

**Q: 支持哪些 AI 提供商？**
A: 主要基于 Claude（Anthropic），但 Team 模式支持通过 tmux 调用 Codex（OpenAI）和 Gemini（Google）CLI。

**Q: 状态存在哪里？**
A: `.omc/` 目录下，全部是本地文件，不上传任何数据到外部服务。

---

## 十一、技术栈

| 技术 | 用途 |
|------|------|
| TypeScript | 核心逻辑 |
| esbuild | 打包为 CJS bridge |
| Vitest | 测试框架 |
| MCP SDK | Model Context Protocol 服务器 |
| better-sqlite3 | 本地数据存储 |
| ast-grep | 结构化代码搜索/替换 |
| vscode-languageserver-protocol | LSP 工具实现 |
| Zod | 运行时类型验证 |
| Commander | CLI 框架 |

---

## 十二、版本与发布

- 当前版本：**4.13.6**
- npm 包名：`oh-my-claude-sisyphus`
- 仓库：`github.com/Yeachan-Heo/oh-my-claudecode`
- 许可证：MIT
