# go-claude-skill: Go conventions skill for Claude Code and Codex

An agent skill, packaged as a plugin for Claude Code and OpenAI Codex, that teaches the agent a team's Go service conventions: code style, the required library stack (GORM, NATS, Fiber v3, zerolog, viper and others) and the project directory layout. Install it and the agent writes and reviews Go code by these rules.

**Claude Code**

```bash
claude plugin marketplace add saturngfish/go-claude-skill
claude plugin install go-service-conventions@go-claude-skill
```

**Codex**

```bash
codex plugin marketplace add saturngfish/go-claude-skill
```

Then install **Go Service Conventions** from the plugin directory. See [Install](#install) for the details and a no-plugin option.

## Why

A coding agent writes valid Go by default, but not *our* Go. Without guidance it picks its own libraries, puts files wherever seems reasonable, and formats code in ways that fail review. Every such mismatch costs a review round or a manual fix.

This skill puts the conventions where the agent reads them, so that generated and reviewed code already follows them:

- **Fewer review comments.** Formatting rules (blank line before `return`, guard returns, logical blocks) are applied while the code is written.
- **One stack across services.** The agent reaches for GORM, NATS, Fiber v3, zerolog and the rest of the agreed libraries instead of alternatives.
- **Predictable project layout.** New code lands in the right directory (`internal/services`, `internal/repository`, `pkg/models`, ...), so every service looks the same.
- **One source of truth.** The rules live in this repository, in a single `SKILL.md` shared by Claude Code and Codex. Update them here and everyone gets the change.

## What the skill covers

| Area | Contents |
|---|---|
| Code formatting | Blank line before `return`, guard returns, separation of logical blocks, with good and bad examples |
| Stack | Required library for each concern: database, transport, web server, validation, logging, config, JWT, Redis, Kafka, UUID, decimals |
| Project layout | Purpose of every directory under `cmd`, `internal/` and `pkg/`, plus where each kind of code goes |
| Review checklist | A short list the agent runs through when reviewing Go code |

The full text is in [`skills/go-service-conventions/SKILL.md`](skills/go-service-conventions/SKILL.md).

## Install

### Claude Code

In your shell:

```bash
claude plugin marketplace add saturngfish/go-claude-skill
claude plugin install go-service-conventions@go-claude-skill
```

Or inside a Claude Code session:

```
/plugin marketplace add saturngfish/go-claude-skill
/plugin install go-service-conventions@go-claude-skill
```

### Codex

**As a plugin.** Add the marketplace, then install **Go Service Conventions** from the plugin directory (`/plugins` in the Codex CLI, or the Plugins page in the ChatGPT desktop app):

```bash
codex plugin marketplace add saturngfish/go-claude-skill
```

**As a plain skill, without plugins.** Copy the skill folder into your user skills directory and restart Codex:

```bash
git clone https://github.com/saturngfish/go-claude-skill
mkdir -p ~/.agents/skills
cp -r go-claude-skill/plugins/go-service-conventions/skills/go-service-conventions ~/.agents/skills/
```

To share it with everyone working in one repository instead, copy the folder into that repository's `.agents/skills/`.

## Usage

The agent loads the skill on its own when a task involves writing, editing, reviewing or scaffolding Go service code. To invoke it explicitly:

| Agent | Explicit call |
|---|---|
| Claude Code | `/go-service-conventions:go-service-conventions` |
| Codex | `$go-service-conventions` |

Example prompts:

- "Add a withdraw endpoint to this service."
- "Review this PR against our Go conventions."
- "Scaffold a new service called `payments`."

## Repository structure

```
.claude-plugin/marketplace.json         # Claude Code marketplace catalog
.agents/plugins/marketplace.json        # Codex marketplace catalog
plugins/
  go-service-conventions/
    plugin.json                         # portable plugin manifest
    .claude-plugin/plugin.json          # Claude Code manifest
    .codex-plugin/plugin.json           # Codex manifest
    skills/go-service-conventions/
      SKILL.md                          # the conventions (shared by both agents)
      agents/openai.yaml                # Codex display metadata
```

## Updating the rules

1. Edit `SKILL.md`.
2. Bump `version` in all three manifests: `plugin.json`, `.claude-plugin/plugin.json` and `.codex-plugin/plugin.json`.
3. Run `claude plugin validate --strict .` and push.

Users pick up the new version with `claude plugin marketplace update go-claude-skill` (Claude Code) or `codex plugin marketplace upgrade` (Codex).
