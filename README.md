# rfo — Repo Forge Orchestrator

<div align="center">
  <img src="rfo_illustration.webp" alt="rfo — GitHub-first multi-repo orchestration for humans and agents">
</div>

<div align="center">

![Platform](https://img.shields.io/badge/platform-Linux%20%7C%20macOS%20%7C%20Windows-blue.svg)
![Rust](https://img.shields.io/badge/Rust-1.85%2B-orange.svg)
![License](https://img.shields.io/badge/License-MIT-yellow.svg)
![Release](https://img.shields.io/github/v/release/quangdang46/repo_forge_orchestrator?include_prereleases)

</div>

**GitHub-first multi-repo orchestration for humans and AI agents.**  
Track many repos in SQLite, keep working copies synced, surface what needs attention, and run plan → apply → rollback automations with safety gates and JSON output.

<div align="center">

```bash
curl -fsSL "https://raw.githubusercontent.com/quangdang46/repo_forge_orchestrator/main/install.sh?$(date +%s)" \
  | bash
```

</div>

---

## 🤖 Agent Quickstart (Robot Mode)

```bash
# Fleet health
rfo health --format json

# Status of all tracked repos
rfo status --format json

# Ranked attention
rfo inbox --format json

# Plan automation
rfo review plan --format json
```

**Output conventions**
- stdout = structured data (JSON)
- stderr = diagnostics, warnings
- exit 0 = success

---

## TL;DR

### The Problem

Managing dozens of GitHub repos by hand fails in predictable ways:

| Pain | Symptom |
|------|---------|
| Drift | Working copies behind / dirty / conflicted |
| Noise inbox | Issues/PRs/CI mixed without ranking |
| Blind automation | Scripts mutate without a reviewable plan |
| Agent shell chaos | Models run raw `git`/`gh` with no safety gates |
| No audit trail | “What did we change last Tuesday?” |

### The Solution

**rfo** is a Rust CLI that orchestrates many repos from one local SQLite source of truth:

| Capability | Command surface |
|------------|-----------------|
| Track & sync | `add` · `import` · `sync` · `status` · `prune` |
| Attention | `health` · `inbox` |
| Risky ops | `review plan` · plan → apply → rollback |
| Safety | secret scan · denylist · quality gates |
| Agents | `--format json` · `robot-docs` · MCP-friendly structure |

> **Status:** early development (v0.2.x) — public API and flags may shift before v1.0.

### Why Use rfo?

| Feature | What it does |
|---------|--------------|
| **Fleet inventory** | One SQLite DB for all tracked repos |
| **First-class sync** | ff-only / rebase / merge, parallel, resume, autostash |
| **Ranked attention** | Health scores + inbox instead of tab soup |
| **Plan-then-apply** | Review/sweep flows produce plans before mutation |
| **Agent-ready JSON** | Structured reads for coding agents and MCP |
| **Doctor + self-update** | Diagnose, repair, upgrade in place |

---

### Quick Example

```bash
rfo init
rfo add quangdang46/repo_forge_orchestrator
rfo import --org my-org --limit 50
rfo sync -j 4
rfo status --format json
rfo health
rfo inbox
rfo doctor
```

---

## Design Philosophy

1. **GitHub-first, local source of truth.**  
   Remote is GitHub; authority for *what we track and did* is local SQLite.

2. **Plan before mutate.**  
   Review, sweep, and train-style flows should produce an inspectable plan before apply.

3. **Agents get JSON, not scraped TUI.**  
   Prefer `--format json` and `robot-docs` over parsing human text.

4. **Safety gates over clever scripts.**  
   Secret scan, denylist paths, and quality checks beat “trust the model with raw git.”

5. **Degrade cleanly.**  
   Absent optional tools must not produce silent half-applies.

---

## How rfo Compares

| Approach | Sync | Attention | Safe automation | Agent-ready |
|----------|------|-----------|-----------------|-------------|
| Manual `gh`/`git` | Manual | Manual | No | Fragile |
| Ad-hoc scripts | Partial | No | Rarely | Opaque |
| IDE multi-root | UI-only | Partial | No | Weak CLI |
| **rfo** | First-class | Ranked inbox | Plan/apply | JSON + MCP |

**When to use rfo:**
- You maintain a fleet of GitHub repos (personal monorepo farm, org mirror, agent lab)
- You want agents to sync/status/health without raw destructive git
- You need an audit trail of runs and plans

**When rfo might not be ideal:**
- Single-repo day-to-day work (plain `git`/`gh` is enough)
- Non-GitHub hosts as primary (secondary support only)
- Fully offline environments without API/git network

---

## Installation

### Linux / macOS

```bash
curl -fsSL "https://raw.githubusercontent.com/quangdang46/repo_forge_orchestrator/main/install.sh?$(date +%s)" | bash
```

Detects platform, downloads the release archive, **verifies SHA256**, installs to `~/.local/bin`.

| Variable | Default | Purpose |
|----------|---------|---------|
| `RFO_VERSION` | `latest` | Pin tag, e.g. `v0.2.0` |
| `RFO_INSTALL_DIR` | `$HOME/.local/bin` | Binary destination |
| `RFO_NO_VERIFY` | unset | `1` skips checksum (avoid) |
| `RFO_FORCE` | unset | `1` overwrites silently |

### Windows (PowerShell 5.1+)

```powershell
irm https://raw.githubusercontent.com/quangdang46/repo_forge_orchestrator/main/install.ps1 | iex
```

Installs `rfo.exe` to `%LOCALAPPDATA%\Programs\rfo` and updates user `PATH`.

### From source

```bash
git clone https://github.com/quangdang46/repo_forge_orchestrator.git
cd repo_forge_orchestrator
cargo build --release
./target/release/rfo --version
```

Requires Rust **1.85+**.

---

## Quick Start

```bash
rfo init
rfo add quangdang46/repo_forge_orchestrator
rfo import repos.list                 # or: rfo import --stars / --org ORG / --user USER
rfo sync
rfo status
rfo health
rfo inbox
rfo doctor
```

### Robot / JSON surface

```bash
rfo health --format json
rfo status --format json
rfo list --format json
rfo robot-docs commands
rfo robot-docs quickstart
```

Prefer structured reads over scraping TUI/text when driving agents.

---

## Commands

```text
rfo [--config-dir <DIR>] [--state-dir <DIR>] [--quiet] [--verbose] [--non-interactive] <COMMAND>
```

| Group | Command | What it does |
|-------|---------|--------------|
| Setup | `init` · `doctor [--fix]` | Config + SQLite; diagnose/repair |
| Repos | `add` · `remove` · `list` · `import` · `prune` | Track inventory |
| Sync | `sync` · `status` · `health` · `inbox` | Working copies + attention |
| Runs | `run list/show/timeline` | Inspect past runs |
| Conflicts | `conflict list/explain/abort/mark-resolved` | Merge/rebase recovery |
| Review | `review plan` (+ apply/rollback flows) | Plan-then-apply |
| Sweep | `sweep …` | Commit / agent sweep helpers |
| Config | `config` | Show / set configuration |
| Meta | `self-update` · `robot-docs` · `fork …` | Upgrade, machine docs, forks |

```bash
# Inventory
rfo add owner/repo
rfo import --org my-org --limit 100
rfo list --owner my-org --format json

# Sync fleet
rfo sync --strategy ff-only -j 8 --autostash
rfo sync --dry-run
rfo sync --resume

# Attention
rfo health --format json
rfo status my-org/service-a
rfo inbox

# Safety-oriented automation
rfo review plan
rfo doctor --fix
rfo self-update --check
```

Run `rfo --help` / `rfo <cmd> --help` for full flags.

---

## Safety Model

| Gate | Default |
|------|---------|
| Secret scan | On before risky apply |
| Denylist paths | Blocks dangerous globs |
| Quality checks | Configurable |
| Plan-first | Review/sweep produce plans before mutation |
| Non-interactive | `--non-interactive` never prompts |

Absent tools degrade cleanly where the design allows — never silent half-applies.

---

## Configuration & State

| Path | Purpose |
|------|---------|
| Config dir | `$XDG_CONFIG_HOME/rfo` (override: `--config-dir`) |
| State dir | `$XDG_STATE_HOME/rfo` (override: `--state-dir`) |
| SQLite | Inventory, runs, health — under state dir |

```bash
rfo init
rfo config
rfo doctor
```

---

## Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│ CLI (crates/rfo)                                            │
│  init · add · sync · health · review · robot-docs · …       │
└────────────────────────────┬────────────────────────────────┘
                             │
     ┌───────────────────────┼───────────────────────┐
     ▼                       ▼                       ▼
┌──────────┐          ┌────────────┐          ┌────────────┐
│ rfo-git  │          │ rfo-github │          │ rfo-sync   │
│ local vc │          │ API / gh   │          │ strategies │
└────┬─────┘          └─────┬──────┘          └─────┬──────┘
     │                      │                       │
     └──────────────────────┼───────────────────────┘
                            ▼
                   ┌────────────────┐
                   │ rfo-state      │
                   │ SQLite source  │
                   │ of truth       │
                   └────────┬───────┘
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
        rfo-review    rfo-jobs/sweep   rfo-mcp / output
        plan/apply    run timeline     agent surfaces
```

Workspace highlights: `rfo-core`, `rfo-config`, `rfo-state`, `rfo-git`, `rfo-github`, `rfo-sync`, `rfo-review`, `rfo-sweep`, `rfo-mcp`, `rfo-provider`, …

---

## Troubleshooting

### `rfo: command not found`

```bash
curl -fsSL "https://raw.githubusercontent.com/quangdang46/repo_forge_orchestrator/main/install.sh?$(date +%s)" | bash
export PATH="$HOME/.local/bin:$PATH"
rfo --version
```

### Auth / GitHub API errors

Ensure `gh auth status` works (or the token env your install expects). `rfo doctor` reports common misconfigurations:

```bash
gh auth status
rfo doctor
rfo doctor --fix
```

### Sync conflicts

```bash
rfo conflict list
rfo conflict explain <id>
# resolve in the working tree, then:
rfo conflict mark-resolved <id>
# or abort:
rfo conflict abort <id>
```

### Interrupted parallel sync

```bash
rfo sync --resume
```

### Checksum verification failed

```bash
# Retry with cache-bust; avoid RFO_NO_VERIFY unless debugging
curl -fsSL "https://raw.githubusercontent.com/quangdang46/repo_forge_orchestrator/main/install.sh?$(date +%s)" | bash
```

---

## Limitations

### What rfo Doesn't Do (Yet)

- **Not a full IDE** — orchestrates repos; does not replace review judgment
- **GitHub-first** — other hosts are secondary
- **Pre-v1.0** — flags and schemas may change

### Known Limitations

| Capability | Current state | Notes |
|------------|---------------|-------|
| Multi-host VCS | ⚠️ Secondary | GitHub is the primary path |
| Network-free mode | ⚠️ Limited | Sync/import need API + git |
| Pixel-perfect TUI | ❌ | CLI + JSON first |
| Fully autonomous merge | ❌ | Plan/apply still needs human/agent policy |

---

## FAQ

### vs plain `gh`?

`gh` is one-repo oriented. `rfo` tracks a fleet, ranks attention, and gates automation.

### Safe for agents?

Prefer JSON reads + plan commands. Do not give bare destructive git without review plans. Use `--non-interactive` in automation.

### Where is state?

Local SQLite under the configured state directory (`rfo init` / `rfo doctor`).

### Can I import stars / orgs?

```bash
rfo import --stars --limit 100
rfo import --org my-org
rfo import --user someuser --limit 50
rfo import repos.list
```

### How do I upgrade?

```bash
rfo self-update --check
rfo self-update
```

---

## About Contributions

Please don't take this the wrong way, but I do not accept outside contributions for any of my projects. I simply don't have the mental bandwidth to review anything, and it's my name on the thing, so I'm responsible for any problems it causes; thus, the risk-reward is highly asymmetric from my perspective. I'd also have to worry about other "stakeholders," which seems unwise for tools I mostly make for myself for free. Feel free to submit issues, and even PRs if you want to illustrate a proposed fix, but know I won't merge them directly. Instead, I'll have Claude or Codex review submissions via `gh` and independently decide whether and how to address them. Bug reports in particular are welcome. Sorry if this offends, but I want to avoid wasted time and hurt feelings. I understand this isn't in sync with the prevailing open-source ethos that seeks community contributions, but it's the only way I can move at this velocity and keep my sanity.

---

## License

MIT (see [LICENSE](LICENSE)). Workspace metadata also allows `MIT OR Apache-2.0` for crate publishing flexibility.

---

<div align="center">

**Many repos. One orchestrator. Plan before apply.**

</div>
