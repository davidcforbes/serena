# Bug report draft — rust-analyzer unbounded memory (40+ GB) on large Rust workspaces

> **Status 2026-10-16:** upstream merged the `cachePriming` part of this fix
> (`3be211c6`, 2026-06-08) and then re-enabled `cargo.autoreload`
> (`1f133f62`, 2026-07-29). The local fix therefore still lives as a commit on
> `local/main`, rebased onto upstream `main`: "rust-analyzer: skip the
> whole-workspace startup flycheck". It sets `cargo.autoreload=False` and
> `check.workspace=False`, and `test/solidlsp/rust/test_rust_analyzer_settings.py`
> asserts both. The branch named below no longer exists.
>
> Original note: draft for an upstream issue/PR to oraios/serena. The fix is the
> companion commit on this branch (`rustback/lightweight-rust-analyzer`).

## Summary

On a large, actively-built Rust workspace, the rust-analyzer process serena
launches repeatedly grew to **40–44 GB RAM** — with **no editor (VS Code/JetBrains)
running**; serena's MCP server was the only thing keeping rust-analyzer engaged.

## Environment

- serena `1.5.4.dev0` (git main), Windows 11, MCP server via `uvx --from git+…/serena`.
- Workspace: a multi-crate Rust workspace (~10 crates) with a path-dependency on a
  forked `rustic_core` and a sibling `d2d-ui` crate — a large dependency graph.
- The workspace is **built continuously** from the terminal (`cargo build/test`
  gate runs) while serena is connected.

## Root cause

`src/solidlsp/language_servers/rust_analyzer.py` launches rust-analyzer with
VS Code's full heavyweight `initializationOptions`, hardcoded, with no knob to
tune them down. Two of these are not needed by serena's symbol tools
(`find_symbol`, `find_referencing_symbols`, `get_symbols_overview`, …) and are
exactly what drives the resident memory on a big workspace:

| Option | Value | Effect |
|---|---|---|
| `cachePriming.enable` | `true` | Eagerly indexes the **entire workspace + all dependencies** at startup — the dominant contributor to the 40 GB resident set |
| `cargo.autoreload` | `true` | **Re-indexes on every `target/` / Cargo metadata change** — so each external `cargo build` triggers a re-index storm |

`checkOnSave` was also examined, but it is **kept enabled** — see below.

## Impact

- 40–44 GB rust-analyzer processes with no editor present.
- Continuous re-indexing whenever the project is built outside serena.
- No configuration option to mitigate; the heavy options are hardcoded.

## Proposed fix (this branch)

Run rust-analyzer in a **symbol-navigation-oriented lightweight profile** by
disabling the two eager/auto re-indexing drivers, while leaving diagnostics
intact:

```python
"cachePriming": {"enable": False, "numThreads": 0},   # was True
"cargo": { "autoreload": False, ... },                # was True
# checkOnSave is intentionally LEFT True (see note)
```

### Why `checkOnSave` must stay enabled

It is tempting to also disable `checkOnSave` for the lightweight profile, but
doing so **breaks serena's `get_diagnostics_for_file` for Rust**. rust-analyzer's
diagnostics for ordinary errors (e.g. an unresolved name → `E0425`) are produced
by flycheck (`cargo check`); with `checkOnSave: false` serena's pull-diagnostics
request comes back empty. This is covered by
`test/solidlsp/rust/test_rust_diagnostics.py::test_file_diagnostics`, which fails
with `[]` when `checkOnSave` is off and passes when it is on. `checkOnSave` only
spawns transient `cargo check` child processes — it is **not** a driver of
rust-analyzer's resident RSS — so keeping it on preserves diagnostics at no
meaningful memory cost.

Ideally upstream would expose these as a config toggle (e.g.
`language_server_options` per language, or a global `lightweight`/`symbol_only`
flag) rather than hardcoding, so users who want full RA features can opt in.

## Repro

1. Open serena on a large multi-crate Rust workspace (no editor running).
2. Build the workspace repeatedly from a terminal (`cargo build`/`cargo test`).
3. Watch `rust-analyzer.exe` RSS climb to tens of GB and re-index on each build.
