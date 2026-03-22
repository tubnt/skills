# PMA + CLAUDE.md Prompt Guide

Practical prompt patterns for the five most common development scenarios.

## Prerequisites

```bash
# Install PMA skill
npx skills add tubnt/skills --skill pma --global

# Install global config (if repo is private, use gh CLI instead of curl)
gh api repos/tubnt/skills/contents/CLAUDE.md -q '.content' | base64 -d > ~/.claude/CLAUDE.md
# Or if repo is public:
# curl -sL https://raw.githubusercontent.com/tubnt/skills/main/CLAUDE.md -o ~/.claude/CLAUDE.md
```

---

## Scenario 1: New Project — Requirements Discussion → Proposal

### Step 1: Kickoff discussion (no code yet)

```
我要做一个 [项目描述]。

目标用户：[谁]
核心功能：
1. [功能1]
2. [功能2]
3. [功能3]

技术约束：[语言/框架/部署方式]

先不要写代码，帮我梳理：
- 系统架构（哪些模块、怎么通信）
- 数据模型（核心实体和关系）
- API 边界（对外暴露什么）
- 需要做的技术决策
```

### Step 2: Refine and output requirements doc

```
基于刚才的讨论，生成 docs/requirements.md（需求文档）。
格式：按功能模块分章节，每个功能写清楚：
- 用户故事
- 验收标准
- 优先级（P0/P1/P2）
```

### Step 3: Initialize PMA and start first task

```
/pma

用 PMA 初始化项目文档结构（task/plan/changelog），
然后把需求拆成任务写入 docs/task/index.md。
先从 P0 功能开始，给我第一个任务的方案。
```

> **Key point**: Do NOT jump to code. Use the discussion phase to align on scope, then let PMA's three-phase workflow take over.

---

## Scenario 2: Existing Project — Onboard AI and Generate PMA Docs

### Step 1: Let AI explore the codebase

```
这是一个已有项目，你先全面了解它：

1. 读项目结构和主要文件
2. 理解架构（模块划分、数据流、技术栈）
3. 读 git log 了解近期变更
4. 找出核心入口点和关键文件

不要改任何代码，只输出你的理解。
```

### Step 2: Generate architecture doc

```
基于你的理解，生成 docs/architecture.md，包含：
- 系统架构图（用 text/mermaid）
- 模块职责划分
- 数据流
- 关键技术决策
- 已知的技术债
```

### Step 3: Initialize PMA tracking

```
/pma

初始化 PMA 文档结构（task/plan/changelog）。
扫描代码中的 TODO/FIXME/HACK 注释，加上 git log 中未完成的工作，
整理成任务列表写入 docs/task/index.md。
```

### Step 4: Populate project memory (if using .claude/memory/)

```
把你对这个项目的理解写入项目根目录下的 .claude/memory/：
- MEMORY.md — 项目概述索引
- backend-patterns.md — 后端规范和踩坑
- frontend-patterns.md — 前端规范和踩坑

只写模型猜不到的项目特有知识（踩坑、架构决策、部署细节），
不要写能从代码推断的东西（文件结构、函数签名等）。
```

> **Key point**: Give AI exploration time before asking it to produce anything. The quality of generated docs depends on how well it understands the codebase.

---

## Scenario 3: Adding a New Feature

### Simple feature (< 3 files)

```
/pma 在 [模块] 中新增 [功能描述]。

参考 [现有相似功能的路径] 的实现模式。
```

PMA will automatically: investigate → propose → wait for your approval → implement.

### Complex feature (>= 3 files or cross-module)

```
/pma 新增功能：[功能描述]

需求背景：[为什么要做这个]
验收标准：
1. [条件1]
2. [条件2]

约束：
- 不要改 [不该动的模块]
- 兼容现有 [API/数据结构]
```

### Approving and starting implementation

After PMA outputs the proposal (Phase 2), you have three choices:

```
# Approve and start implementation
proceed

# Approve with minor adjustments
proceed，但 [具体调整]

# Request changes to the proposal
[你的反馈/问题]
```

### Tips for better results

| Technique | Example |
|-----------|---------|
| **Reference existing patterns** | `参考 src/api/orders.go 的实现模式` |
| **Constrain scope** | `只改后端，前端我自己处理` |
| **Specify non-goals** | `不要重构现有代码，只新增` |
| **Provide test criteria** | `完成后用 curl 验证这个接口能返回正确数据` |

### What PMA does for you

You just write a one-line request. PMA handles:
1. **Phase 1**: Reads related code, traces call chains, finds/creates task
2. **Phase 2**: Outputs proposal (current state → changes → risks → scope), waits for your OK
3. **Phase 3**: Implements, verifies, updates task/plan/changelog

> **Key point**: The more context you give (background, constraints, references), the better the Phase 2 proposal. But you don't need to tell PMA _how_ to investigate — it knows.

---

## Scenario 4: Fixing a Bug

### With clear reproduction steps

```
/pma Bug：[症状描述]

复现步骤：
1. [步骤1]
2. [步骤2]
期望：[应该发生什么]
实际：[实际发生什么]
错误信息：[贴日志/截图路径]
```

### With only a symptom (no clear cause)

```
/pma 排查问题：[症状描述]

已知信息：
- 环境：[生产/测试/本地]
- 何时开始出现：[时间/最近的变更]
- 影响范围：[哪些用户/功能]

先调查根因，不要急着修。
```

### With error logs

```
/pma 线上报错：

[粘贴错误日志]

帮我定位根因。
```

### Tips for bug fixing

| Technique | When to use |
|-----------|-------------|
| **Paste the exact error** | Always — don't paraphrase error messages |
| **Mention recent changes** | If the bug started after a deploy/merge |
| **Constrain the fix** | `最小化修复，不要顺便重构` |
| **Ask for verification** | `修完后告诉我怎么验证` or `跑一下相关测试` |
| **Reference the file** | `问题可能在 src/scanner/tron.go 的 processTransfer 方法` |

> **Key point**: PMA will create a BUG-NNN task, investigate, propose a fix, and wait for your approval. You don't need to diagnose the bug yourself — just provide symptoms and context.

---

## Scenario 5: Parallel Development with BKD

Use BKD (AI agent task engine) to split a feature into multiple tasks and execute them in parallel with isolated worktrees. Best for large features that can be decomposed into independent sub-tasks.

### Step 1: Plan and split tasks with PMA

```
/pma 新增功能：[功能描述]

帮我拆分成可以并行开发的独立子任务，每个子任务要求：
- 改动范围不重叠（不同文件/模块）
- 可以独立验证
- 明确的验收标准
```

### Step 2: Create BKD project (first time only)

```
用 BKD 创建项目 [项目名]，工作目录设为当前项目根目录。
```

### Step 3: Dispatch tasks to BKD agents

```
把 docs/task/index.md 中的 [任务范围] 分发到 BKD 执行。
每个任务用独立 worktree（useWorktree=true），避免冲突。

流程：
1. create-issue（statusId=todo）
2. follow-up-issue 发送详细需求和约束
3. update-issue（statusId=working）触发执行
```

### Step 4: Monitor and review

```
检查 BKD 所有任务的执行状态。
对已完成的任务（review 状态）查看 logs，确认质量后移到 done。
```

### BKD workflow diagram

```
PMA Plan (Phase 2)
  ↓ approved
Split into sub-tasks
  ↓
┌─────────────────────────────────────────┐
│  BKD Issue 1        BKD Issue 2         │
│  (worktree-a)       (worktree-b)        │
│  todo → working     todo → working      │
│  → review           → review            │
└─────────────────────────────────────────┘
  ↓ all reviewed
Merge worktrees → Final verification → done
```

### Tips for BKD tasks

| Technique | Why |
|-----------|-----|
| **Always use `useWorktree=true`** | Prevents agents from stepping on each other's changes |
| **`create-issue` with `statusId=todo` first** | Allows you to send detailed follow-up before execution starts |
| **Check `list-processes` before dispatching** | Monitor system load, avoid overloading |
| **One module per issue** | Minimizes merge conflicts between worktrees |
| **Include file paths in follow-up** | Reduces agent investigation time |

### Example: Adding multi-language support

```
# After PMA splits the feature into 3 independent tasks:

用 BKD 并行执行以下任务（都用 worktree）：

1. FEAT-001: 后端 i18n 中间件 + 翻译加载
   - 改动：internal/middleware/i18n.go, internal/config/
   - 验收：API 响应根据 Accept-Language 返回对应语言

2. FEAT-002: 前端 i18n 框架集成 + 中英文翻译文件
   - 改动：pwf-admin/src/i18n/, pwf-admin/src/App.vue
   - 验收：切换语言后所有页面文本正确显示

3. FEAT-003: 支付页多语言 + 自动检测
   - 改动：pwf-pay/src/
   - 验收：支付页根据浏览器语言自动切换

每个任务的 follow-up 要包含具体文件路径和现有代码模式的参考。
```

> **Key point**: PMA handles planning and quality control. BKD handles parallel execution. Use PMA to split tasks intelligently, then dispatch to BKD. After all BKD issues are in `review`, merge and do a final PMA verification pass.

---

## Anti-Patterns to Avoid

| Don't | Do instead |
|-------|-----------|
| `帮我写一个用户管理系统` (too vague) | Break into specific features with acceptance criteria |
| `改一下那个 bug` (no context) | Provide symptoms, errors, reproduction steps |
| `把代码优化一下` (open-ended) | Specify what to optimize and the target metric |
| Skip PMA and jump to code | Always let PMA investigate first — it catches things you'd miss |
| Give PMA step-by-step tool instructions | PMA knows how to use its tools. Just describe _what_ you want |
| Write a 500-word prompt | Keep it concise. Context > verbosity |
| Create BKD issues with `statusId=working` | Always `todo` first, then `follow-up-issue`, then `working` |
| Dispatch overlapping file scopes to BKD | One module per issue to avoid merge conflicts |

---

## Quick Reference

```bash
# Start any task with PMA
/pma [一句话描述任务]

# PMA will ask you to confirm before implementing.
# Type "proceed" to approve.
# Type feedback to refine the proposal.

# After completion, PMA updates:
# - docs/task/index.md (task status)
# - docs/plan/PLAN-NNN.md (if non-trivial)
# - docs/changelog.md (if needed)

# For parallel execution, dispatch to BKD after PMA planning:
# create-issue (todo) → follow-up-issue (details) → update-issue (working)
# Monitor: list-processes / get-issue-logs
# Review: move to "review" → human confirms → "done"
```
