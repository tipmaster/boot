# Fedora Development Environment Setup

Guide for going from a fresh Fedora installation to active development.

**Environment:** fue (Fedora 43) - Testing & development server
**Hostname:** fedora.flywheel.bz
**SSH:** `ssh deploy@fedora.flywheel.bz`

## Hardware Specifications

| Spec | Value |
|------|-------|
| CPU | Intel Xeon E5-1650 v2 @ 3.50GHz |
| Cores/Threads | 6 cores / 12 threads |
| RAM | 125 GB |
| Disk | 937 GB NVMe (870GB available) |
| OS | Fedora 43 |
| Kernel | 6.17.1 |
| Node.js | v22.20.0 |
| Python | 3.14.0 |

---

## 1. Install Claude Code

Claude Code is Anthropic's official CLI tool for AI-assisted development.

### Prerequisites

- Node.js 18+ (comes with npm)

### Installation

```bash
npm install -g @anthropic-ai/claude-code
```

### First Run

```bash
claude
```

On first run, you'll be prompted to authenticate with your Anthropic account.

## 2. Install GitHub CLI (gh)

The GitHub CLI provides GitHub functionality directly from the terminal.

### Installation

```bash
sudo dnf install gh
```

### Authentication

```bash
gh auth login
```

Follow the prompts to authenticate with your GitHub account.

## 3. Clone Projects

All projects are cloned into `/opt`:

```bash
cd /opt
gh repo clone <repo-name>
```

## 4. Configure Development Environment

Use Claude to configure dotfiles, context-engineering, and claude-config for Fedora compatibility:

```bash
cd /opt/dotfiles
claude

cd /opt/context-engineering
claude

cd /opt/claude-config
claude
```

Review and adapt each project's configuration to work with the Fedora environment.

## 5. Install Applications

### VS Code

```bash
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" | sudo tee /etc/yum.repos.d/vscode.repo > /dev/null
sudo dnf install code
```

### Ghostty

```bash
sudo dnf install ghostty
```

### Brave Browser

```bash
sudo dnf install dnf-plugins-core
sudo dnf config-manager addrepo --from-repofile=https://brave-browser-rpm-release.s3.brave.com/brave-browser.repo
sudo dnf install brave-browser
```

---

## Changes Applied for Fedora

The following changes have been extracted from neighboring projects and applied for Fedora compatibility.

### From dotfiles (Shell/ZSH Configuration)

**Server Detection & Environment:**
- Added `IS_SERVER=true` detection for non-Mac hosts
- Server starts in `/opt` directory on login
- Unset `npm_config_prefix` before loading nvm to avoid conflicts
- Use `~/.local/bin/claude` on servers (vs `~/.npm-global/bin/claude` on Mac)

**Claude Aliases (renamed cc* → cl* to avoid C compiler conflict):**
```bash
cl    # Continue session
cln   # New session
clo   # Opus model + continue
clh   # Headless/print mode
clq   # Quiet (no verbose)
cls   # Short session (10 turns)
```

**Server-Specific Claude Wrapper:**
- Uses `/opt/dotfiles/scripts/claude-deploy.sh` wrapper for dangerous mode
- Runs Claude as `deploy` user via `su` (not sudo) to avoid root taint
- Preserves working directory when switching users

**Tmux Layout Aliases:**
```bash
t3    # 3-column layout → /opt/dotfiles/scripts/tmux-three-panes.sh
t6    # 2x3 grid layout → /opt/dotfiles/scripts/tmux-six-panes.sh
t7    # 7 tiled panes   → /opt/dotfiles/scripts/tmux-seven-panes.sh
```

**Worktrunk Integration:**
- Shell integration: `eval "$(wt config shell init zsh)"`
- `wsc` - switch worktree + create + run claude
- `ws` - switch worktree + run claude

**Removed Mac-Only Features:**
- Terminal/Ghostty aliases (Mac-only `open -a` commands)
- Brave browser aliases
- pbpaste clipboard integration
- Homebrew paths

### From context-engineering

**Three-Environment Documentation:**
- Added `fue` (Fedora 43) as testing/development environment
- Updated environments table in CLAUDE.MD and INFRASTRUCTURE.MD
- Added dev column in services availability table

**Infrastructure Changes:**
- Scripts must be kept in project directory (`/opt/<project>/scripts/`)
- Daily auto-upgrade for coding agents (Claude Code, Codex, Gemini CLI)

### From infra-config

**Server Access:**
```bash
# fue (Dev Server)
ssh deploy@fedora.flywheel.bz

# Project Folders
/opt/<project-dir>
```

**Service Availability (fue vs production):**
| Service | fue | Production |
|---------|:---:|:----------:|
| MariaDB/MySQL | - | ✓ |
| MongoDB | - | ✓ |
| ClickHouse | - | ✓ |
| Redis | - | ✓ |
| nginx | - | ✓ |
| PM2 | - | ✓ |

**Note:** fue is a fresh dev environment. Services installed as needed.

**User Migration:**
- SSH access moved from root to deploy user
- Systemd services updated to run as deploy user
- All commands updated to use `deploy@` instead of `root@`

### From bootstrap

**DNF Dependencies Script:**
- Uses `dnf` instead of `yum` for package management
- Includes Development Tools group installation
- Perl, Node.js, Python dependencies
- PHP, MySQL/MariaDB development packages

---

## Project Structure on Fedora

```
/opt/
├── athlete-training/    # Training app
├── boot/               # This bootstrapping repo
├── bootstrap/          # Installation scripts
├── claude-config/      # Claude configuration
├── context-engineering/ # Shared AI context files
├── dotfiles/           # Shell, git, editor configs
├── fedora-hacks/       # Fedora-specific customizations
├── infra-config/       # Infrastructure documentation
├── network-config/     # Network configuration
├── project-mgmt/       # Project management
├── promos/             # Promotions app
├── tmapp/              # Tipmaster app
├── tmapp-responsive-mobile/
└── tmdata/             # Tipmaster data
```

---

## Symlink Structure

Most projects share common context files via symlinks to `context-engineering`:

```
project/
├── AGENTS.MD → ../context-engineering/AGENTS.MD
├── CLAUDE.MD → ../context-engineering/CLAUDE.MD
├── GEMINI.MD → ../context-engineering/GEMINI.MD
├── INFRASTRUCTURE.MD → ../infra-config/INFRASTRUCTURE.MD
├── SHARED_INSTRUCTIONS.MD → ../context-engineering/SHARED_INSTRUCTIONS.MD
└── TOOLS.MD → ../context-engineering/TOOLS.MD
```

---

## Quick Start After Fresh Clone

```bash
# 1. Source the dotfiles
ln -sf /opt/dotfiles/shell/zshrc ~/.zshrc
source ~/.zshrc

# 2. Start working
cd /opt
t7                  # Open 7-pane tmux layout
cl                  # Start Claude in first pane
```
