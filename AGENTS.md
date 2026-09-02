This project uses Serena for development.
Make sure that you read the project's `critical_info` memory before working on it.
* When using Serena, the memory is provided either
  - along with the result of the `initial_instructions` tool, or
  - upon activating the project with `activate_project`.
  Be sure to always call the `initial_instructions` tool!
* When not using Serena, read `./.serena/memories/critical_info.md` manually.  

## Local Serena CLI and Codex

On this Windows workstation, use the repository virtual environment as the
canonical Serena runtime. Do not install `serena-agent` with `uv tool`.

- `C:\Users\david\bin\serena.exe` must be a copy of
  `C:\dev\serena\.venv\Scripts\serena.exe`. Refresh the copy after recreating
  `.venv` or regenerating its entrypoints.
- Configure the Codex MCP server named `serena` to run
  `C:\dev\serena\.venv\Scripts\serena.exe` with `start-mcp-server`,
  `--project-from-cwd`, and `--context=codex`.
- Preserve `PYTHONUTF8=1`, `PYTHONIOENCODING=utf-8`, and
  `startup_timeout_sec = 60` in the Codex MCP configuration.
- Verify from a different repository with `serena --version` and
  `codex mcp get serena --json`. Restart open Codex clients after changing the
  runtime or MCP configuration.
