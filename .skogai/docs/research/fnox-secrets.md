# fnox and secrets: how the pieces fit (Nov 2026)

Sources: `mise docs` in mise docs checkout (`~/.local/src/mise/docs`) (`environments/secrets/*.md`,
`dotfiles/{managed,encryption}.md`, `bootstrap/secrets.md`), `fnox provider add --help`
(fnox 1.44.0), `mise 2026.11.14`. Provider details beyond the `--help` list were not read.

## Current setup
- `~/.config/fnox/config.toml`: global fnox config (age provider, default). Tracked by dotfiles (`*.toml`).
- `~/.config/fnox/age.txt`: private key. Excluded from dotfiles tracking by the `include` filter. Single copy, no backup.
- `fnox.toml` (this repo): age provider + ciphertext. `[secrets.fnox]` in `.skogai/config/mise.toml` makes mise the consumer.
- Delivery: only a task listing `secrets = [...]`, or `mise x --secrets KEY -- cmd`, gets a value. Shell, `mise env`, hooks do not.

## Where each mise feature fits
| Need | Use |
|---|---|
| Secret for a task / one command | fnox via `[secrets.fnox]` (above) |
| A secret *file* deployed by dotfiles (`~/.npmrc`, tokens) | `[dotfiles] "<path>" = { mode = "fnox" }`; store with `fnox set --file <path>` (fnox >= 1.44) |
| A tracked file that must be in history but encrypted | `[history.encryption] recipients = [...]` + `encrypt = true` on the entry; uses mise's own age identity (`~/.config/mise/age.txt`, `age.key_file`), separate from fnox's |
| A secret rendered into a template | `[bootstrap.secrets]` + `{{ secret(name=...) }}`; value comes from an env var (e.g. `fnox exec -- mise dot apply`) |
| Currently excluded credential files (`anthropic/credentials`, `gh/hosts.yml`, `sunshine/credentials`, `gptme/oauth`, ...) | candidates for `mode = "fnox"`: backed up and portable without plaintext in history |

## Ansible Vault
Not a fnox provider. The `fnox provider add` type list is: 1password, age, aws, aws-kms, aws-ps, azure-ac,
azure-kms, azure-sm, gcp, gcp-kms, fido2, bitwarden, doppler, enpass, foks, gopass, bitwarden-sm, infisical,
keepass, keeper-sm, keychain, kubernetes, lua, password-store, passwordstate, plain, proton-pass, vault, yubikey.
`vault` there is HashiCorp Vault, not ansible-vault. Options: a `lua` provider plugin wrapping `ansible-vault view`
(needs the `ansible-vault` binary, not installed on this machine), or re-encrypt the data into fnox/age and let
Ansible read values via `fnox exec`. The old vault password files are not needed once secrets live in fnox.

## Alternatives to fnox + age (all in the mise/fnox docs)
- SOPS file (`.env.yaml`, ...): decrypted into *all* `[env]`, including the shell. Less scoped than fnox.
- mise age values in `mise.toml`: same exposure as SOPS.
- fnox with another provider: `password-store`/`gopass` (GPG), `keychain`, `bitwarden`, `1password`, `fido2`/`yubikey` (hardware).
- Untracked `mise.local.toml` / `.env`: simplest, no encryption, no sync.

## Open
- Back up `age.txt` and add a recovery recipient (an independent second age key) to the provider.
- Decide whether mise history encryption reuses the fnox key or gets its own.
