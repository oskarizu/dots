# 0001. Keep chezmoi for public dotfiles; secrets and private hosts stay local
- Status: Accepted
- Date: 2026-11-20
- Supersedes: none

## Context
Public repo oskarizu/dots (chezmoi) went stale since 2024 while the live shell config evolved. The repo is public and previously received a DMCA takedown for a committed font.

## Decision
Keep chezmoi. Track zsh, vim, git, gh (config only), thefuck, git-nav, starship, an ssh config template and a global Brewfile. Personal data (name, email, GPG id) lives only in local chezmoi.toml via promptStringOnce. Secrets (~/.secrets), private hosts (~/.ssh/config.local, ~/.zshrc.local), gh hosts.yml and all keys are never tracked and are documented in the README. Fonts are blocked by .gitignore/.chezmoiignore and a CI check; gitleaks runs in CI over full history.

## Alternatives considered
- yadm / bare git repo: loses templating and externals.
- Encrypted secrets in repo (age/1Password): unnecessary complexity for one small secrets file.

## Consequences
New machines need Homebrew, then a one-liner, then the private local files created by hand. oh-my-zsh is an exact external, so its cache moves to ~/.cache/oh-my-zsh.
