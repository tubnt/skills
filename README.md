# Claude Code Skills

个人定制的 Claude Code Skills 和全局配置。

## 安装

### PMA Skill（项目管理助手）

```bash
npx skills add tubnt/skills --skill pma --global
```

### CLAUDE.md（全局配置）

手动复制到 `~/.claude/CLAUDE.md`：

```bash
gh api repos/tubnt/skills/contents/CLAUDE.md --jq '.content' | base64 -d > ~/.claude/CLAUDE.md
```

## 包含内容

### pma/
项目开发生命周期管理，三阶段工作流（调查 → 方案 → 实现），文件化任务/方案追踪。

### CLAUDE.md
全局 Agent 配置：中文输出、Git 规范、安全规则、AIssh 服务器操作、问题分级等。
