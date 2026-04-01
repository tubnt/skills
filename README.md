# Claude Code Skills

个人定制的 Claude Code Skills 和全局配置。

## 安装

### 全部 Skill 一键安装

```bash
for skill in pma pma-bun pma-go pma-rust pma-web pma-code-review; do
  npx skills add tubnt/skills --skill $skill --global
done
```

### 单独安装

```bash
npx skills add tubnt/skills --skill pma --global
npx skills add tubnt/skills --skill pma-bun --global
npx skills add tubnt/skills --skill pma-go --global
npx skills add tubnt/skills --skill pma-rust --global
npx skills add tubnt/skills --skill pma-web --global
npx skills add tubnt/skills --skill pma-code-review --global
```

### CLAUDE.md（全局配置）

手动复制到 `~/.claude/CLAUDE.md`：

```bash
gh api repos/tubnt/skills/contents/CLAUDE.md --jq '.content' | base64 -d > ~/.claude/CLAUDE.md
```

## 包含内容

### pma/
项目开发生命周期管理，三阶段工作流（调查 → 方案 → 实现），文件化任务/方案追踪。

### pma-bun/
Bun + TypeScript 后端实现基线。技术栈：Hono 框架、Drizzle ORM + PostgreSQL、pino 日志、Zod 验证。

### pma-go/
Go 后端实现基线。技术栈：net/http + Chi 路由、sqlc + pgx 数据访问、koanf 配置、slog 日志。

### pma-rust/
Rust 后端实现基线。技术栈：Tokio + Axum、Diesel/SQLx、figment 配置、rustls-only TLS。

### pma-web/
React 19 前端实现基线。技术栈：Vite 8、TanStack Router/Query、Zustand、shadcn/ui + Tailwind CSS 4.2。

### pma-code-review/
栈感知代码审查。支持本地差异审查、PR 审查（直接发布 GitHub 评论）、仓库审计三种模式，自动检测技术栈加载对应审查规则。

### CLAUDE.md
全局 Agent 配置：中文输出、Git 规范、安全规则、AIssh 服务器操作、问题分级等。
