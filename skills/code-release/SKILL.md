---
name: code-release
description: Pre-release audit for research code repos. Walks through secrets/credentials, dependencies & conda envs, documentation, licensing, code quality, data/artifacts, reproducibility, SLURM/launcher scripts, CI/testing, and repository hygiene before a public release. Geared to GPU/SLURM/conda research codebases with vendored forks.
---

# Code Release

## Overview

Research codebases routinely leak secrets through committed `.env` files, ship `environment.yml` that doesn't create from clean state, hardcode cluster-specific paths, and drag scratch sweep outputs into release commits. By the time a paper drops and reviewers clone the repo, those problems become public-facing — sometimes in ways that can't be quietly fixed (rotated keys, rewritten history, retracted artifacts).

This skill is the audit pass that catches those problems while they're still local-only. It walks 10 categories of release-prep rules, each with concrete checks. The rules come from real incidents: secrets leaking because `.env.template` was incomplete, conda envs failing because they pinned a vendored fork that `PYTHONPATH` already shadowed, sbatch scripts silently emitting `--account=` when an env var was unset.

## When to Use

- Preparing a private research repo for public release (paper companion, tool announcement).
- Tagging a new minor/major version of an already-public research repo.
- Inheriting an existing repo for release where the current state has not been audited.
- Before any public-mirror push when upstream is a private repo.

## When NOT to Use

- Internal-only scripts that will never leave the team's cluster.
- Weekend prototypes with no users.
- Hot-fix patches to an already-released repo where the audit was done recently and only one file changed.

## Audit Checklist

Walk each section in order. Each rule is imperative and individually checkable; failures block release until fixed.

### Secrets & Credentials

- `.env` must be in `.gitignore` — never commit actual API keys or machine-specific secrets.
- Ship a `.env.template` with every key the code reads AND every key present in any local `.env`, using placeholder values (`your-api-key`, `/path/to/x`). Verify with:
  ```bash
  comm -23 \
    <(grep -oE '^[A-Z_]+=' .env | sort -u) \
    <(grep -oE '^#?\s*[A-Z_]+=' .env.template | sed 's/^#\s*//' | sort -u)
  ```
  Empty output means the template is in sync. Keys with sensible in-code defaults can stay commented out.
- `.env.template` must not leak cluster-specific identifiers (partition names, account tags, QoS labels). Use generic placeholders (`gpu`, `cpu`) and comment out optional fields with a short purpose note.
- Before acting on any "secret exposed" alert (rotation, `git filter-repo`, history rewrite), verify with `git log --all --full-history -- <path>`. "Secret present in file on disk" ≠ "secret committed to git". Audit tools routinely miss this distinction; rewriting history on a shared branch is expensive and disruptive if the premise was wrong.

### Dependencies & Environment

- `environment.yml` must be creatable from a clean state — test `conda env create -f environment.yml` end-to-end before release.
- Remove pip pins for packages loaded via `PYTHONPATH` (vendored forks), since they fail to resolve or silently shadow the fork at runtime.
- Packages that can't install in one pass get a footer block documenting the post-install step. Keep footers instructions-only. Common categories:
  - GPU kernel packages with build-from-source quirks (e.g. `flash-attn` — ship a prebuilt wheel from GitHub releases since the `setup.py` has a cross-device rename bug, and building from source needs `CUDA_HOME`).
  - Version conflicts between two libraries (e.g. `transformers==5.x` + `vllm` — vllm pins `transformers<5`, so install `transformers` after the env is created).

### Documentation

- When a repo splits a pipeline across multiple top-level folders (e.g. training vs. evaluation), each folder gets its own README and `.env.template` scoped to its concerns. Cross-link the folders and explicitly call out shared vs. separate conda envs so users know whether to recreate the env or activate the existing one.
- README must state GPU type and count for the main experiments (e.g. "A100 80GB ×4 for the largest model evaluated"). Lets users decide whether the repo is runnable for them before they debug CUDA OOMs.
- If the repo ships trained weights or other release artifacts, the README must state where they live (in-repo path or HuggingFace hub ID). If nothing is released, no statement is needed — do not add a placeholder "not released" section.

### Licensing & Attribution

- Ship a `LICENSE` file at repo root. Without one, the default under copyright law is "all rights reserved" — users technically can't use the code even if the README implies it's open.
- Vendored code (e.g. a vendored framework directory) must preserve the upstream `LICENSE` and any `NOTICE` / attribution files inside the vendored tree. Pinning a fork doesn't strip the license obligation.

### Code Quality

- Docstring usage examples that reference filesystem paths must use `.env` vars (e.g. `"$ARTIFACT_BASE_DIR"`) and show a `source .env` preamble, not hardcoded absolute paths.
- Conda env variable name is `CONDA_ENV` (not `CONDA_ENV_NAME`) across every `.env.template` and every script that activates an env.

### Data & Artifacts

- Large output/dataset directories stay out of git: add to `.gitignore` and `git rm -r --cached <path>` if already tracked.
- Ignore artifact directories via glob patterns (`sbatch_results*/`, `outputs_*/`, `results_*/`), not version-specific literals. Version suffixes proliferate with sweep iterations; each new suffix slips past a literal-name ignore.

### Reproducibility

- Conda env names must not be hardcoded in scripts. Source `.env` first, then `conda activate "$CONDA_ENV"` (or use `${CONDA_ENV:?Set CONDA_ENV in .env}` to fail loudly).
- Absolute paths must not be hardcoded in scripts or configs. Use `.env` + `${VAR}` expansion; mirror defaults in `.env.template`.
- `.env` must be sourced *before* `conda activate` in any script that reads `$CONDA_ENV`.

### SLURM / Launcher Scripts

- Guard optional env vars in sbatch command construction: `${SLURM_ACCOUNT:+--account=$SLURM_ACCOUNT}` produces zero arg elements when unset. Unguarded `--account=${SLURM_ACCOUNT}` produces `--account=` (empty value), which most clusters reject.
- `sbatch --dependency` only honors the last flag. Combine multiple predecessors into one flag with colon-separated IDs: `--dependency=afterok:J1:J2`. Emitting two `--dependency` flags silently drops all but the last.

### CI / Testing

_Add rules here as CI/testing learnings accumulate._

### Repository Hygiene

- Rename commits should be pure — don't mix content edits into a rename, or `git log --follow` / `git blame` break at the boundary. Commit content changes first, then do the rename as its own commit.
- Large restructures: `git mv` tracked files, plain `mv` untracked files, stage only what should be committed.
- `.gitignore` updates are not committed as part of release work — change them in the working tree only. Use explicit `git add <paths>` over `git add .` to avoid sweeping `.gitignore` (or stray untracked files) into release commits.

## Common Pitfalls

- **"Secret on disk" ≠ "secret in git history".** Audit tools that scan the working tree don't always check `git log`. Verify with `git log --all --full-history -- <path>` before rewriting history; a rewrite on a shared branch is expensive and disruptive if the premise was wrong.
- **Empty-value sbatch args.** Unguarded `--account=${SLURM_ACCOUNT}` produces `--account=` when the var is unset; most clusters reject it. Always use `${VAR:+--flag=$VAR}`.
- **`sbatch --dependency` last-flag-only.** Emitting `--dependency=afterok:J1` and `--dependency=afterok:J2` silently drops the first. Combine: `--dependency=afterok:J1:J2`.
- **Literal-name `.gitignore` rules.** A rule for `outputs_v3/` does not catch `outputs_v4/`. Use globs: `outputs_*/`. Sweep iterations proliferate suffixes.
- **Pip-pinning a vendored fork.** If a vendored framework directory is on `PYTHONPATH`, pip-pinning the same package upstream either fails resolution or silently shadows the fork at runtime. Remove the pin.

## Integration with Other Skills

- Pair with `implement-review` to run a staged-change code review on the release-prep commit itself.
- Pair with `readme-polish` after the README state checks trigger a polish pass on the README.
