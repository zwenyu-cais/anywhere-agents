# anywhere-agents

**A portable, battle-tested AI agent stack. Bootstrap, evolve, personalize.**

`anywhere-agents` is a maintained, opinionated configuration for AI coding agents — a production stack that turns Claude Code, Codex, and related tools into coherent infrastructure across every project, every machine, every session.

The name has two meanings:

- **Portability** — the stack runs anywhere you work. Any OS, any repo, any machine where your AI agents operate.
- **Openness** — the stack is battle-tested by one practitioner and published for anyone who wants a well-tested starting point they can bootstrap, evolve, and personalize into their own. A launchpad, not a framework.

## Status

**Pre-release.** This repository is reserved for the upcoming v1.0 launch. Source will appear here shortly.

## What it will include

- **Skills** — production-grade protocols including `implement-review` (a structured dual-agent review loop with round history and content-type lenses), `dual-pass-workflow`, `bibref-filler`, `figure-prompt-builder`, `ci-mockup-figure`
- **Safety hook** — `guard.py`, a PreToolUse hook that has caught real destructive commands in daily use. Deliberately memorable warnings ("STOP! HAMMER TIME!") make the friction impossible to auto-dismiss
- **Settings** — curated permissions, hook wiring, effort-level defaults for Claude Code and Codex
- **Agent instructions** — battle-tested writing defaults, formatting rules, Git safety, shell command style, environment conventions
- **Bootstrap** — a shell script that syncs the whole stack into any project repo, idempotent across sessions

## Distribution

Two adoption paths will be supported:

1. **Consume directly** — one-line bootstrap points at `yzhao062/anywhere-agents`, refreshes on every session, customize locally via `AGENTS.local.md`
2. **Fork and track** — fork this repo, diverge freely, pull upstream via `git pull upstream main` when you want

No CLI install. No YAML manifest. No framework.

## Follow for release

Follow [@yzhao062](https://github.com/yzhao062) for the release announcement.

## License

Apache 2.0.
