# PROJECT

## What it is

`skogai` is a real program, installed by mise like any other tool. Its CLI is
defined by a [usage](https://usage.jdx.dev) spec, so its `--help`, man page
and shell completions come from that spec and are never written by hand.

This repo (`skogai/skogai`) is the hub that installs it and holds the
SkogAI environment around it.

## Use case

A person or an agent clones the hub and runs `mise install`. They get:

- the `skogai` program on `PATH`
- completions for `skogai`: with mise activated, `skogai <Tab>` works with no
  extra setup
- the SkogAI env (`$SKOGAI`, …) in every shell and agent session

Humans and agents use the same interface. An agent reads the usage spec
(`skogai usage`) instead of guessing the arguments.

## How mise delivers it

The hub config pins the program through mise's `packslip:` backend:

```toml
[tools]
"packslip:github.com/skogai/<repo>" = "latest"
```

`packslip:` is the only backend where mise installs completions and man pages
for a tool. It builds completions from a usage spec with its own built-in
engine, so users don't need `usage` installed. See
`.skogai/src/mise/docs/dev-tools/packslip-resources.md`.

## Publishing

A GitHub Actions release job builds the binaries, uploads them to the
release, then runs `jdx/packslip@v1`:

- Signing uses the workflow's OIDC identity through Sigstore, so there are
  no keys to manage. The job needs `contents: write`, `id-token: write` and
  `attestations: write`.
- The `resources` input declares the usage spec, e.g.
  `cli-spec/usage=exec:skogai usage`.

Guide: https://packslip.dev/docs/publishing/

## Basic features

1. `skogai --help`, man page and completions, all generated from one usage
   spec.
2. `skogai usage` prints that spec, for agents and for packslip.
3. Installed and versioned by mise through `packslip:`.
4. The hub's `[env]` and `[deps]` set up the surrounding environment.

## Out of scope (for now)

- what the subcommands actually do: decided one at a time later
- which secrets exist and which age recipients encrypt them (fnox is wired into mise, but `fnox.toml` has no providers yet)
- security hardening (comes after it works)

## Open decisions

- the implementation language of `skogai`, and the name of its repo

## Done when

```sh
mise install && skogai --help && mise completion zsh --tool skogai
```

That command installs the program, prints help from its usage spec, and
prints a completion script that mise built from the same spec.
