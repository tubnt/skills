---
name: pma
description: 项目开发生命周期管理，三阶段工作流（调查 → 方案 → 实现），文件化方案/任务追踪，多 Agent 协作支持。用于功能开发、Bug 修复、重构、进度追踪等场景。
---

# PMA - 项目管理助手

以明确的阶段门控、最小化改动、文件化任务/方案追踪驱动交付工作。

## 硬性规则

1. 文档、任务文件、commit message 使用中文。文件名使用英文。
2. 先读后写：修改逻辑前必须检查调用链、相关配置/测试、最近的 changelog。
3. 只做请求的最小改动，不添加未请求的重构或功能。
4. 不使用 plan mode（`EnterPlanMode`），方案管理在 `docs/plan/` 文件中。
5. 未经明确确认（`proceed`）不得开始实现。

## 三阶段工作流

### 第一阶段：调查

1. 追踪上下游调用链、符号引用、类型定义。
2. 搜索相关代码、配置、测试、迁移、文档。
3. 读 `docs/changelog.md` 末尾了解近期上下文。
4. 在 `docs/task/index.md` 中找到或创建对应任务并认领（`[-]`）。

非平凡任务规则：
- 改动涉及 `>=3` 个文件或跨模块时，创建 `docs/plan/PLAN-NNN.md` 并将调查发现写入现状章节。

### 第二阶段：方案

输出以下内容，然后停下等待确认：
- 现状
- 方案
- 风险
- 工作量
- 备选方案（如有多种方案）

非平凡任务：
- 完成 `PLAN-NNN.md` 的剩余章节。
- 在 `docs/plan/index.md` 追加一行 `[ ]`。
- 等待用户批注，处理完所有批注后才进入实现。

### 第三阶段：实现 → 验证 → 记录

确认后执行：

1. 如有方案文件，设置方案索引标记为 `[-]`，详情 `status` 为 `implementing`。
2. 按审批通过的方案逐步实现。
3. 执行聚焦验证（编译、测试、部署验证等）。
4. 设置任务索引标记为 `[x]`，详情 `status` 为 `completed`。
5. 如有方案文件，设置方案索引标记为 `[x]`，详情 `status` 为 `completed`。
6. 按需更新 changelog。

## 任务和方案文件

使用以下规范参考（不在此重复定义格式）：

- 任务格式：[docs/task-format.md](docs/task-format.md)
- 方案格式：[docs/plan-format.md](docs/plan-format.md)

必需结构：

- `docs/task/index.md`：单行任务条目
- `docs/task/PREFIX-NNN.md`：任务详情文件
- `docs/plan/index.md`：单行方案条目
- `docs/plan/PLAN-NNN.md`：方案详情文件

## 认领机制（多 Agent 安全）

在写任何实现代码之前：

1. 读 `docs/task/index.md`；对于 `[-]` 项，读详情中的 `owner`。
2. 如果其他 Agent 拥有进行中的任务，跳过。
3. 原子认领：
   - 更新任务索引 `[ ] -> [-]`
   - 更新任务详情 `status -> in_progress`，设置 `owner`
   - 如有 TaskUpdate 工具可用，调用 `TaskUpdate(status: "in_progress", owner: "<agent>")`
4. 认领完全写入后才开始实现。

完成时：
- 设置任务索引 `[-] -> [x]`
- 设置任务详情 `status -> completed`

关闭/不做时：
- 设置任务索引为 `[~]`
- 设置任务详情 `status -> closed` 并记录原因

## 同步规则

任务状态更新立即执行，不延迟。

- 主要来源是 `docs/task/` 和 `docs/plan/` 中的文件。
- 如有 `TaskCreate`/`TaskUpdate` 工具可用，保持工具状态与文件状态同步。
- 如任务工具不可用，仅使用文件同步，并在进度更新中说明。

会话检查清单：
1. 会话开始：读 `docs/task/index.md`、活跃任务详情、`docs/plan/index.md`。
2. 新任务：先创建详情文件，再追加索引行。
3. 开始工作前：完成认领流程。
4. 会话结束：验证状态已写入，更新索引头部日期。

## Changelog 规范

条目格式：

```markdown
## YYYY-MM-DD HH:MM [标签]

[内容]
```

推荐标签：
- `[进度]`、`[BUG-P0]`、`[BUG-P1]`、`[踩坑]`、`[决策]`

## 项目初始化

首次在项目中使用时：

1. 确保 `docs/task/index.md` 存在（从 [docs/task-format.md](docs/task-format.md) 初始化）。
2. 确保 `docs/plan/index.md` 存在（从 [docs/plan-format.md](docs/plan-format.md) 初始化）。
3. 确保 `docs/changelog.md` 存在。

## PR 工作流

### 创建 PR

1. 分析从分支点开始的**完整** commit 历史，不仅是最新 commit。
2. 使用 `git diff [base-branch]...HEAD` 审查所有改动。
3. 标题：70 字符以内。
4. Body 格式：
   ```
   ## 概要
   <1-3 个要点>

   ## 测试计划
   - [ ] <检查项>
   ```
5. 新分支 push 时使用 `-u` 标志。

### PR 前自动审查

创建或更新 PR 前自动执行：

1. **代码审查** — 审查所有改动文件。
2. **安全扫描** — 检查硬编码密钥、输入校验、注入漏洞、错误信息泄露。
3. **构建验证** — 确保构建通过。
4. **测试** — 运行测试套件，验证无回归。

任何检查失败时先修复再创建 PR，不使用 `--no-verify` 跳过。
