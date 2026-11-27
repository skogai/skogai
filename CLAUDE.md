# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`skogai/skogai` is the root/hub repo for the SkogAI setup (`$SKOGAI` = `/home/skogix/skogai`). It holds almost no code of its own yet. It is a young repo (a handful of commits) being rebuilt from a minimal base, so treat everything as early-stage and add things only when asked.

## Layout and conventions

- Repo-specific material lives under `.skogai/`, not the repo root:
  - `.skogai/config/`: tool config. `.skogai/config/mise.toml` is the real mise config.
  - `.skogai/docs/`: docs (e.g. `docs/research/lod.md`). Keep using this location instead of adding a top-level `docs/`.
  - `.skogai/src/`: upstream sources vendored as git submodules (currently `.skogai/src/mise` → `git@github.com:skogai/mise.git`, branch `main`).
- mise only reads `.skogai/config/mise.toml` when `MISE_DEFAULT_CONFIG_FILENAME=.skogai/config/mise.toml` is already in the environment before mise starts. On this machine it is set in `~/.config/environment.d/50-skogai-mise.conf` (needs a re-login). The same line in the root `mise.toml` `[env]` only reaches child processes, not mise's own config discovery. Put tool/task/deps changes in `.skogai/config/mise.toml`, not the root `mise.toml`.
- Tasks for agents carry the tag `agent` (our own convention; mise gives tags no meaning). Find them with `mise tasks ls --tag agent --json`, run them with `mise run --tag agent`.
- `.claude/settings.json` has a `SessionStart` hook that writes `eval "$(mise hook-env -s zsh)"` to `$CLAUDE_ENV_FILE`, so Claude Code's Bash commands get the project's mise tools and `[env]`.
- `config/config.json` is the legacy SkogAI config: per-agent env blocks (`skogai`, `claude`, `dot`, `amy`, `goose`, `codex`, …), CLI `settings`, and the `$` block defining SkogAI notation. Other tools read it via `$SKOGAI_CONFIG`. It contains credentials, so never print, copy, or move its values anywhere.

## Commands

```sh
mise config ls                      # verify .skogai/config/mise.toml is listed
mise deps                           # [experimental] runs [deps]; git-submodule auto-inits submodules
git submodule update --init --recursive   # manual alternative
```

There is no build, lint, or test setup in this repo. `.skogai/src/mise` is a full Rust project with its own `AGENTS.md`. Read that file before working inside the submodule, and commit there, not here.

## Direction

The stated goal in `.skogai/config/mise.toml` is to install the SkogAI CLI through mise (`"pipx:skogai/cli" = "latest"`, currently commented out). Context and docs for agents follow the plan in `.skogai/docs/research/lod.md`: keep context files short and hand-written with only facts an agent can't work out itself, route at most one level deep, and generate symbol outlines instead of writing them by hand.
