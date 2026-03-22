## Agent Rules

- All output (docs, code comments, commit messages, PR descriptions) MUST be in Chinese by default.
- Communicate with the user in Chinese.
- Do not mention AI assistants, agents, or collaborator model names in any remote-visible content.

## Project Preferences

- If a project uses the *PMA skill* for management, strictly follow the PMA workflow (investigate -> proposal -> implement). Do not skip phases or bypass the file-based task tracking process.

## Documentation Rules

- Do not create unnecessary doc files (e.g. summary.md, report.md). Temporary files go to `./tmp/`.
- Frontend/backend API docs live in code as source of truth. When adding/modifying APIs, update both sides together.
- Human-authored project docs (requirements, design, etc.) are maintained by the user. AI may add implementation status annotations only with confirmation.

## Technical Preferences

- Deployment priority: docker compose > docker run > bare metal.
- Compile locally, deploy artifacts to server. **Never compile on remote servers.**

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

## Git

- Commit messages, PR titles, PR descriptions, and all remote-visible Git metadata MUST be in Chinese.
- Conventional commits format: `<type>: <Chinese description>` (type in English: feat, fix, refactor, docs, test, chore, perf, ci).
- Do not mention AI assistants or model names (Codex, Claude, ChatGPT, OpenAI, Anthropic, etc.) in any remote-visible content.

## Security

1. All secrets/passwords/tokens go in `.env`, never hardcoded. Update `.env.example` when adding new ones.
2. Never log or document secrets or passwords. Report security vulnerabilities immediately.
3. **No weak passwords** — always use `openssl rand -base64 24` or equivalent to generate strong random passwords.

## Large File Handling

- When reading large files, use the Read tool's offset and limit parameters for chunked reads.
- Prefer Grep to search for specific content rather than reading entire files.
- For files over 1000 lines, use Grep to locate target regions before reading chunks.

## UI Component Rules

All UI interactive components (Dialog, Dropdown, Select, Tabs, Tooltip, Popover, etc.) must use mature headless UI libraries. Do not hand-write focus trap, scroll lock, ARIA management, or keyboard navigation. Only implement manually when no library or community alternative exists, and document the reason in a comment.

## AIssh Server Operations

All remote server operations (deploy, restart, config, logs, Docker, etc.) **MUST go through AIssh MCP**. Do not use local Bash ssh/scp/rsync.

### Workflow
1. Run `servers_list` to get available servers and their notes.
2. Match target server by notes (production/test/build). Ask user to confirm if notes are unclear.
3. Use `exec_run` for commands / `file_deploy` for files / `file_fetch` for downloads.
4. `exec_run` requires a `reason` field.
5. `file_deploy` requires staging first: `curl -s -F 'file=@<local_path>' -H 'Authorization: Bearer <token>' https://<domain>/mcp/upload` to get `file_ref`, then call `file_deploy`.

### Prohibited
- **Never compile on remote servers** — compile locally, deploy artifacts via `file_deploy`.
- **Never write files via exec_run** (e.g. `echo ... | base64 -d > file`). Use `file_deploy` for file transfers.
- Confirm with user before running destructive commands.

## Issue Severity

| Level | Definition | Action |
|-------|------------|--------|
| P0 | Production outage / security vulnerability | Report immediately, wait for confirmation |
| P1 | Core feature broken | Report with proposal, wait for confirmation |
| P2 | Minor feature issue | Auto-fix |
| P3 | UX improvement | Auto-fix |
