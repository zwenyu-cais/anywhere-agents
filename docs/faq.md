# FAQ

??? question "Does this work with [agent X]?"
    Primary support is **Claude Code + Codex**. The `AGENTS.md` convention is standardized enough that other agents (Cursor, Aider, Gemini CLI) may read it and pick up writing defaults. Skill routing and guard hooks are tuned for Claude Code specifically. Forks can extend support to other agents.

??? question "What is the difference between `AGENTS.md` and `AGENTS.local.md`?"
    `AGENTS.md` is the shared config synced from upstream. Bootstrap overwrites it on every run — never edit it in a consuming project, or your changes will be lost on the next session.

    `AGENTS.local.md` is your per-project override. Bootstrap never touches it. Use it for project-specific permissions, domain glossaries, or opt-outs from shared defaults.

??? question "How do I disable the guard hook?"
    In a fork, remove the user-level section of `bootstrap/bootstrap.sh` and `bootstrap/bootstrap.ps1` that deploys `scripts/guard.py` to `~/.claude/hooks/`. Then repoint your consumers at your fork.

    In a specific project only, remove the `hooks` entry from `~/.claude/settings.json` manually — but bootstrap will re-install it on the next run unless you have also removed it from the fork.

??? question "Why does `git push` always ask for confirmation?"
    The `Git Safety` section of `AGENTS.md` says _"Never run `git commit` or `git push` without explicit user approval."_ This is a deliberate opinion. The guard hook enforces it: even if Claude Code has permissions set to auto-approve `git push`, the hook intercepts and requires explicit confirmation.

    To disable, remove the Git Safety section from your fork's `AGENTS.md` and the corresponding rules from `scripts/guard.py`.

??? question "Why is `anywhere-agents` on both PyPI and npm?"
    Agent-native installs. Users can tell their agent _"install anywhere-agents in this project"_ and the agent picks whichever command matches the environment (`pipx run`, `npx`, or raw shell). The PyPI and npm packages are thin shims — both download the same shell bootstrap and run it. There is no Python / Node.js logic in the install path itself.

??? question "Can I use this without the shell bootstrap?"
    Yes — manually copy `AGENTS.md`, `skills/`, `scripts/guard.py`, and the `.claude/` / `user/` settings into your project. The bootstrap is a convenience wrapper, not a requirement.

??? question "How do I update across many projects at once?"
    Bootstrap runs on every session and pulls from upstream, so every consuming project updates automatically on its next session. No manual per-project maintenance.

    To force a refresh mid-session in one project, run `bash .agent-config/bootstrap.sh` (or `& .\.agent-config\bootstrap.ps1` on Windows).

??? question "How do I debug a skill that is not dispatching?"
    Check `my-router`'s lookup order:

    1. `skills/<name>/SKILL.md` in the project (project-local override).
    2. `.agent-config/repo/skills/<name>/SKILL.md` (bootstrapped copy).
    3. Installed agent-platform plugins (e.g., Claude Code plugin skills).

    If the skill exists but is not dispatching, verify the routing rules in `skills/my-router/references/routing-table.md`. The router prefers keyword matches over file-type matches; a too-generic keyword can accidentally match the wrong skill.

??? question "Is this maintained?"
    Yes — it is the author's daily-driver config. Changes land when the author needs them. Bug fixes and documentation improvements are accepted via PR. Feature requests that do not match the author's work should land in a fork.

??? question "What does the version number mean?"
    `anywhere-agents` uses [Semantic Versioning](https://semver.org). Repo tags, PyPI, and npm all share one version stream — a tag like `v0.1.2` reproduces exactly what is on the package registries.

    - **Major (`0.x.y → 1.0.0`)**: the user-facing install flow or config contract changes.
    - **Minor (`0.1.x → 0.2.0`)**: new shipped skills or user-visible features.
    - **Patch (`0.1.0 → 0.1.1`)**: documentation, packaging, or hygiene changes that do not change behavior.

    While in 0.x, "minor" is used loosely per SemVer's 0.x convention.

??? question "Where can I report bugs or propose changes?"
    - Bugs and clear fixes → [GitHub Issues](https://github.com/yzhao062/anywhere-agents/issues) or PR.
    - Feature requests that do not match the author's workflow → fork and maintain your own version; pull upstream fixes as they land.
    - Documentation improvements → always welcome via PR.

    See [CONTRIBUTING.md](https://github.com/yzhao062/anywhere-agents/blob/main/CONTRIBUTING.md).
