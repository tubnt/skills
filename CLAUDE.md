## Agent Rules

- 所有输出（文档、代码注释、commit message、PR 描述）默认使用中文。
- 与用户沟通使用中文。
- 禁止在任何远程可见内容中提及 AI 助手、Agent 或协作者模型名称。

## Project Preferences

- If a project uses the *PMA skill* for management, strictly follow the PMA workflow (investigate -> proposal -> implement). Do not skip phases or bypass the file-based task tracking process.

## Shell
- Prefer `bash` for all command execution.
- Do not use `zsh` unless the user explicitly requests it.
- When a tool supports explicit shell selection, set it to `bash`.
- *NEVER* use `kill $(lsof -ti:PORT)` without filtering — it kills ALL processes on that port, including Claude Code itself. If you must kill by port, use `kill $(lsof -ti:PORT -sTCP:LISTEN)` or `fuser -k PORT/tcp`.

## Tmux Server Management
- *ALWAYS* use tmux to run dev servers, test servers, and long-running processes.
- Session name: `{project_dir_basename}-{path_hash}` — generate with:
  ```bash
  echo $(basename "$PWD" | tr '.' '-')-$(echo -n "$PWD" | md5sum | cut -c1-6)
  ```
- *Before starting*: check `tmux has-session -t NAME` first — reuse or restart existing sessions.
- *NEVER* kill ports as the first approach — manage process lifecycle through tmux.
- See `/work/ai/rules/tmux-servers.md` for detailed workflow.

## Git
- Commit messages、PR titles、PR descriptions 等 Git 元数据使用中文。
- Conventional commits 格式：`<type>: <中文描述>`（type 用英文：feat, fix, refactor, docs, test, chore, perf, ci）。
- 禁止在 commit、PR、评论等远程可见内容中提及 AI 助手或模型名称（Codex、Claude、ChatGPT、OpenAI、Anthropic 等）。

## 安全

1. 所有密钥/密码/Token 存入 `.env`，禁止硬编码；新增时同步更新 `.env.example`
2. 绝不在文档或日志中记录密钥、密码。发现安全漏洞立即报告
3. **禁止弱密码**——任何场景都必须使用 `openssl rand -base64 24` 或等效方式生成随机强密码

## 大文件处理

- 读取大文件时，使用 Read 工具的 offset 和 limit 参数分段读取
- 优先使用 Grep 搜索特定内容，而非读取整个文件
- 单个文件超过 1000 行时，先用 Grep 定位目标区域再分段读取

## UI 组件规范

所有 UI 交互组件（Dialog、Dropdown、Select、Tabs、Tooltip、Popover 等）必须使用成熟的 headless UI 库实现，禁止从零手写。不要手写 focus trap、scroll lock、ARIA 属性管理、键盘导航等底层逻辑。仅当库中确实没有对应组件且无社区替代时才允许手动实现，并在注释中说明原因。

## AIssh 服务器操作规则

所有涉及远程服务器的操作（部署、重启、配置、日志、Docker 等），**必须通过 AIssh MCP 执行**，禁止通过本地 Bash 的 ssh/scp/rsync 直接连接。

### 操作流程
1. 先执行 `servers_list` 获取可用服务器及其 notes 备注
2. 根据 notes 匹配目标服务器（生产/测试/构建）；notes 不清晰时先向用户确认
3. `exec_run` 执行命令 / `file_deploy` 部署文件 / `file_fetch` 下载文件
4. `exec_run` 的 `reason` 字段必须填写
5. `file_deploy` 需先暂存：`curl -s -F 'file=@本地路径' -H 'Authorization: Bearer <token>' https://<域名>/mcp/upload` 获取 `file_ref`，再调用 `file_deploy`

### 禁止事项
- **禁止在远程服务器上编译**——所有编译在本地完成，通过 `file_deploy` 部署产物
- **禁止通过 exec_run 写入文件**（如 `echo ... | base64 -d > file`），文件传输必须用 `file_deploy`
- 破坏性命令执行前先向用户确认

## 问题分级

| 级别 | 定义 | 处理 |
|------|------|------|
| P0 | 生产故障/安全漏洞 | 立即报告，等确认 |
| P1 | 核心功能异常 | 报告方案，等确认 |
| P2 | 次要功能问题 | 自动修复 |
| P3 | 体验优化 | 自动修复 |
