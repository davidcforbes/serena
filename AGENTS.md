This project uses Serena for development.
Make sure that you read the project's `critical_info` memory before working on it.
* When using Serena, the memory is provided either
  - along with the result of the `initial_instructions` tool, or
  - upon activating the project with `activate_project`.
  Be sure to always call the `initial_instructions` tool!
* When not using Serena, read `./.serena/memories/critical_info.md` manually.  

## Local Serena CLI and Codex

On this Windows workstation, the canonical Serena runtime is the `uv tool`
install of `serena-agent` from this repository. Do not use the repository
`.venv` (if present) as the runtime; it exists only for running tests.

- Entry point: `C:\Users\david\.local\bin\serena.exe` (shim managed by `uv tool`).
- After changing Serena code in this repo, reinstall the tool so the shim runs
  the new code, e.g. `uv tool install --force -e .` (or reinstall
  `serena-agent`). Newly started MCP servers pick up the new code immediately;
  already-running servers keep the old code in memory until the client restarts.
- Codex MCP server named `serena` in `C:\Users\david\.codex\config.toml` runs
  `C:/Users/david/.local/bin/serena.exe` with `start-mcp-server`,
  `--context=codex`, and `--project-from-cwd`.
- Preserve `PYTHONUTF8=1`, `PYTHONIOENCODING=utf-8`, and
  `startup_timeout_sec = 90` in that configuration. The current 90 s headroom
  covers slow first starts right after a tool reinstall.
- Verify from a different repository with `serena --version` and
  `codex mcp get serena --json`. Restart open Codex/Claude clients after
  changing the runtime or MCP configuration.
