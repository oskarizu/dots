# dots

macOS dotfiles managed with [chezmoi](https://www.chezmoi.io/).

## What is tracked

- zsh: `.zshrc` (template) and `.zshenv`
- git: `.gitconfig` (template) with delta and commit signing
- starship, vim, thefuck and git-nav config
- Rex terminal config: `~/.config/rex/init.lua`
- gh config (not `hosts.yml`, which holds credentials)
- `~/.ssh/config`: a minimal template that `Include`s an untracked `~/.ssh/config.local`
- `~/.Brewfile`: a global Brewfile, plus a `run_onchange_` script that runs
  `brew bundle --global --no-upgrade` whenever the Brewfile changes
- oh-my-zsh and 4 third-party plugins, fetched as chezmoi externals
  (`.chezmoiexternal.toml`, pinned to commit SHAs and bumped manually). oh-my-zsh is `exact`, so its
  cache lives in `~/.cache/oh-my-zsh` via `ZSH_CACHE_DIR` in `.zshrc`.

## Apps whose settings live outside this repo

Rex replaces Ghostty as the terminal; its config is tracked here
(`~/.config/rex/init.lua`).

Raycast: `~/.config/raycast` is deliberately NOT tracked. It only holds
installed Store extensions (third-party code, about 800 MB, re-downloaded from
the Store), not configuration.

- On a new Mac, reinstall these extensions from the Raycast Store: Spotify
  Player, Linear, 1Password, Google Chrome.
- Raycast's real settings (hotkeys, quicklinks, snippets) live in the app. Back
  them up with Raycast's built-in "Export Settings & Data" (Settings -> Advanced,
  or the Export Settings & Data command), which produces a password-protected
  `.rayconfig` file.
- Keep that file somewhere private, never in this public repo (`*.rayconfig` is
  gitignored). Import it on the new Mac via Raycast's "Import Settings & Data".

## Bootstrap on a new Mac

Install [Homebrew](https://brew.sh/) first, then:

```sh
brew install chezmoi && chezmoi init --apply oskarizu/dots
```

Alternative without Homebrew-installed chezmoi:

```sh
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply oskarizu/dots
```

On first run chezmoi prompts once for your email, full name and GPG key id.
The answers are stored only in `~/.config/chezmoi/chezmoi.toml`, not in this
repo.

Prerequisites that are not in the Brewfile:

- gnupg and pinentry-mac: commit signing is on (`brew install gnupg pinentry-mac`)
- git-lfs: the gitconfig requires the lfs filter (`brew install git-lfs`)
- fzf: optional, fzf-zsh-plugin is only loaded when fzf exists (`brew install fzf`)
- starship: the prompt (`brew install starship`)
- VS Code: `EDITOR` is `code -w` (`brew install --cask visual-studio-code`)

Review before applying:

```sh
chezmoi init oskarizu/dots
chezmoi diff
chezmoi apply
```

Caveat: the Brewfile includes cargo and npm entries. On a fresh machine
Rust and Node must be installed first, so the brew-bundle step may report
errors at the end of the first apply. Install them (for example rustup and
nvm) and re-run `brew bundle --global`.

## Required untracked files

Create these by hand. Never commit them.

- `~/.secrets` (mode 600): defines `NOTION_TOKEN` and `NOTION_DAILY_LOG_DB_ID`
- `~/.zshrc.local`: machine-specific aliases, sourced before SDKMAN
- `~/.ssh/config.local` (mode 600): private `Host` blocks, `Include`d first by `~/.ssh/config`
- `~/.config/gh/hosts.yml`: created by `gh auth login`
- GPG and SSH keys: import or generate separately; keys are never tracked

## Day-to-day workflow

- Edit: `chezmoi edit <file>`, or edit the source and run `chezmoi apply`
- Inspect: `chezmoi diff` and `chezmoi status`
- Add a file: `chezmoi add <file>` (use `--template` for machine-specific bits)
- Pull updates: `chezmoi update`
- Commit and push from the source dir: `chezmoi cd`
- Health checks: `chezmoi verify` and `chezmoi doctor`

## Rules and safety

- This repo is public. Never add secrets, tokens, private hostnames or keys.
- CI runs gitleaks and a font check on every push.
- **No fonts.** Never commit font files (ttf, otf, woff, woff2, ttc, dfont).
  A previous repo name was taken down by a DMCA complaint because a licensed
  font was committed. `.gitignore` and `.chezmoiignore` block them and CI
  fails if one is tracked. Install fonts via Homebrew casks instead (font
  casks are not currently in the Brewfile).

- **Custom zsh goes outside `~/.oh-my-zsh`.** The oh-my-zsh external is
  `exact`, so anything placed in `~/.oh-my-zsh/custom` is deleted on the next
  apply. Keep custom zsh in `~/.zshrc.local` or in a `$ZSH_CUSTOM` directory
  outside `~/.oh-my-zsh`.

## Updating pinned externals

The externals are pinned to commit SHAs with sha256 checksums, so a compromised
upstream branch cannot run code on your machines. To bump one:

1. Get the new commit: `gh api repos/<owner>/<repo>/commits/HEAD --jq .sha`
2. Read the upstream diff since the pinned SHA:
   `gh api repos/<owner>/<repo>/compare/<old>...<new> --jq .html_url`
3. Download `https://github.com/<owner>/<repo>/archive/<new>.tar.gz` and compute
   `shasum -a 256` of it.
4. Update `url` and `checksum.sha256` in `.chezmoiexternal.toml`.
5. Run `chezmoi diff`, then `chezmoi apply`.

GitHub does not formally guarantee byte-stable archive checksums. If a checksum
mismatch ever appears for an unchanged SHA, re-verify the archive contents
before updating the hash.

## Layout

| Source name | Target |
| --- | --- |
| `dot_` prefix | dotfile (`dot_zshenv` -> `~/.zshenv`) |
| `private_` prefix | restrictive permissions (`private_dot_ssh` -> `~/.ssh`) |
| `.tmpl` suffix | rendered as a template |
| `run_onchange_after_` | script run after apply when its content changes |

`README.md`, `LICENSE`, `docs` and `.github` are repo-only and never applied
to `$HOME` (see `.chezmoiignore`).

Design rationale: [docs/decisions/0001-keep-chezmoi-public-dotfiles.md](docs/decisions/0001-keep-chezmoi-public-dotfiles.md)
