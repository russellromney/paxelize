# paxelize

Stage local agent history so Paxel can find more of it.

Paxel already understands several agent history formats. The rough edge is that
some tools put useful sessions outside Paxel's default search paths. `paxelize`
copies those supported stores into one staging tree and prints the environment
variables Paxel needs.

## What It Stages

- Claude JSONL transcripts from `~/.claude/transcripts`
- Claude Code project transcripts from `~/.claude/projects`, when present
- Codex CLI and Codex Desktop rollouts from `~/.codex/sessions`
- Archived Codex rollouts from `~/.codex/archived_sessions`
- opencode sessions from `~/.local/share/opencode/opencode.db`

It does not copy auth files, arbitrary app caches, browser stores, or raw source
repositories.

## Install

```bash
git clone https://github.com/russellromney/paxelize.git
cd paxelize
```

Optional:

```bash
export PATH="$PWD/bin:$PATH"
```

## Use

Preview what would be staged:

```bash
./bin/paxelize doctor
```

Stage history:

```bash
./bin/paxelize stage
```

Run Paxel against the staged copy:

```bash
./bin/paxelize run
```

Run Paxel across every detected project:

```bash
./bin/paxelize run -- --all
```

Or print the Paxel command and run it yourself:

```bash
./bin/paxelize env
```

Check the currently staged copy:

```bash
./bin/paxelize counts
```

## Where Files Go

By default, `paxelize stage` creates:

```text
~/.paxel/all-agent-history-YYYYMMDD-HHMMSS
```

It also updates:

```text
~/.paxel/all-agent-history-current
```

Paxel is then pointed at:

```text
CLAUDE_DIR=~/.paxel/all-agent-history-current/claude
CODEX_DIR=~/.paxel/all-agent-history-current/codex
OPENCODE_DIR=~/.paxel/all-agent-history-current/opencode
```

## Options

```text
paxelize stage [--stage-dir DIR] [--current-link DIR] [--dry-run]
paxelize counts
paxelize run [--paxel-url URL] [-- PAXEL_ARGS...]
paxelize env
paxelize doctor
paxelize clean
```

Environment variables:

- `PAXELIZE_STAGE_DIR`: exact stage directory to create
- `PAXELIZE_CURRENT`: stable symlink path
- `PAXELIZE_PAXEL_URL`: Paxel upload script URL

## Requirements

- Bash
- `rsync`
- `sqlite3` for opencode DB snapshots
- `curl` for `paxelize run`

If `sqlite3` is missing, Claude and Codex staging still work. opencode is
skipped with a warning.

## Safety Notes

`paxelize` copies local transcript stores into a staging directory. It does not
upload anything by itself during `stage`, `doctor`, or `env`.

`paxelize run` downloads and runs Paxel's upload script with the staged
environment variables set. Review Paxel's script and data handling policy before
running it if you have not already.
