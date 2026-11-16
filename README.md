# go-claude-skill: Go conventions skill for Claude Code

A Claude Code plugin (skill) that teaches Claude a team's Go service conventions: code style, the required library stack (GORM, NATS, Fiber v3, zerolog, viper and others) and the project directory layout. Install it and Claude writes and reviews Go code by these rules.

```bash
claude plugin marketplace add saturngfish/go-claude-skill
claude plugin install go-service-conventions@go-claude-skill
```

## Why

Claude writes valid Go by default, but not *our* Go. Without guidance it picks its own libraries, puts files wherever seems reasonable, and formats code in ways that fail review. Every such mismatch costs a review round or a manual fix.

This skill puts the conventions where Claude reads them, so that generated and reviewed code already follows them:

- **Fewer review comments.** Formatting rules (blank line before `return`, guard returns, logical blocks) are applied while the code is written.
- **One stack across services.** Claude reaches for GORM, NATS, Fiber v3, zerolog and the rest of the agreed libraries instead of alternatives.
- **Predictable project layout.** New code lands in the right directory (`internal/services`, `internal/repository`, `pkg/models`, ...), so every service looks the same.
- **One source of truth.** The rules live in this repository; update them here and everyone gets the change.

## What the skill covers

| Area | Contents |
|---|---|
| Code formatting | Blank line before `return`, guard returns, separation of logical blocks, with good and bad examples |
| Stack | Required library for each concern: database, transport, web server, validation, logging, config, JWT, Redis, Kafka, UUID, decimals |
| Project layout | Purpose of every directory under `cmd`, `internal/` and `pkg/`, plus where each kind of code goes |
| Review checklist | A short list Claude runs through when reviewing Go code |

The full text is in [`plugins/go-service-conventions/skills/go-service-conventions/SKILL.md`](plugins/go-service-conventions/skills/go-service-conventions/SKILL.md).

## Install

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

## Usage

Claude loads the skill on its own when a task involves writing, editing, reviewing or scaffolding Go service code. To invoke it explicitly:

```
/go-service-conventions:go-service-conventions
```

Example prompts:

- "Add a withdraw endpoint to this service."
- "Review this PR against our Go conventions."
- "Scaffold a new service called `payments`."

## Repository structure

```
.claude-plugin/
  marketplace.json                      # marketplace catalog
plugins/
  go-service-conventions/
    .claude-plugin/plugin.json          # plugin manifest
    skills/go-service-conventions/
      SKILL.md                          # the conventions
```

## Updating the rules

1. Edit `SKILL.md`.
2. Bump `version` in `plugins/go-service-conventions/.claude-plugin/plugin.json`.
3. Run `claude plugin validate .` and push.

Users pick up the new version with `claude plugin marketplace update go-claude-skill`.
