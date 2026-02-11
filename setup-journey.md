# OpenClaw Setup Journey — Mac Mini (M-series)

Complete timeline of setting up OpenClaw from scratch on a fresh Mac mini.

## Jan 25, 2026 — Day 1: Initial Install

### Prerequisites
```bash
# Homebrew (fresh Mac)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/opt/homebrew/bin/brew shellenv zsh)"' >> ~/.zprofile

# Git
brew install git

# Oh My Zsh + Powerlevel10k (optional but nice)
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
brew install --cask font-meslo-lg-nerd-font

# Zsh plugins (optional)
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# Package managers
npm install -g pnpm
brew tap oven-sh/bun && brew install bun
```

### ClawdBot Install (original name, now OpenClaw)
```bash
curl -fsSL https://clawd.bot/install.sh | bash
```

### Post-Install Configuration
```bash
clawdbot configure --section web        # Web UI
clawdbot configure --section providers  # API keys (Anthropic, etc.)
clawdbot configure --section models     # Model selection
clawdbot doctor --fix                   # Auto-fix any issues
clawdbot daemon restart
```

### Claude CLI (for Claude Code integration)
```bash
curl -fsSL https://claude.ai/install.sh | bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
claude setup-token
```

### Channel Setup & Heartbeat
```bash
clawdbot channels status --probe
clawdbot config set agents.defaults.heartbeat.every "30m"
clawdbot config set agents.defaults.heartbeat.target "last"
clawdbot daemon restart
```

## Jan 30 — Migration: ClawdBot → OpenClaw

The project rebranded from ClawdBot to OpenClaw:
```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

This migrated configs from `~/.clawdbot/` to `~/.openclaw/` and renamed the CLI from `clawdbot` to `openclaw`.

## Jan 31 — Skills & Extensions

### Installing Skills
```bash
npx molthub@latest install moltbook    # Moltbook social network skill
```

## Feb 2 — Security & Tailscale

### Security Audit
```bash
openclaw security audit --deep
```

### Tailscale (Remote Access)
Initially tried the brew formula, but the macOS cask version works better:
```bash
# Don't do this:
# brew install tailscale && brew services start tailscale

# Do this instead:
brew install --cask tailscale
# Then open Tailscale.app and sign in
tailscale up
```

## Feb 8 — Auth Refresh

### Re-authenticating Providers
```bash
openclaw models auth login --provider anthropic
openclaw models auth login    # Interactive mode
```

### GitHub CLI
```bash
gh auth login    # Follow interactive prompts
```

---

## Known Issues & Gotchas

### ⚠️ Gateway "Service Not Loaded" After Update

**Problem:** After running `openclaw update`, the launchd service can become unloaded, resulting in "Gateway service not loaded" errors. The update replaces the npm package but does NOT re-register the launchd plist.

**Symptoms:**
- `openclaw gateway restart` fails with "service not loaded"
- Gateway won't start
- All channels (WhatsApp, Discord, etc.) go offline

**Fix:**
```bash
openclaw gateway install    # Re-writes the launchd plist
openclaw gateway restart    # Loads and starts the service
```

**Recommendation:** Always run `openclaw gateway install` after `openclaw update` as a precaution.

### ⚠️ Playwright Must Be Full Package

**Problem:** Browser automation (X/Twitter posting, web scraping) requires the full `playwright` package. OpenClaw ships with `playwright-core` which doesn't include browser binaries or the full API.

**Symptoms:**
- "Playwright is not available in this gateway build" errors
- Browser `navigate`, `snapshot`, `act` all fail
- `browser start` works (launches Chrome) but can't control it

**Fix:**
```bash
cd /opt/homebrew/lib/node_modules/openclaw
npm install playwright
openclaw gateway install
openclaw gateway restart
```

**Note:** This may need to be re-done after each `openclaw update` since npm updates can reset dependencies.

### ⚠️ Brave Search Rate Limits (Free Plan)

- 1 request/second
- 2,000 requests/month
- Will get 429 errors if exceeded

### ⚠️ Gateway SIGUSR1 Restart vs Full Restart

`openclaw gateway restart` sends SIGUSR1 for a graceful in-process restart. This does **not** reload new npm packages. If you've installed new dependencies (like Playwright), you need a full stop/start:

```bash
openclaw gateway stop
openclaw gateway start
```

Or just use `gateway install` + `gateway restart` which handles it properly.

---

## Daily Maintenance Routine

```bash
openclaw update                          # Update to latest
openclaw gateway install                 # Re-register service (safety net)
openclaw gateway restart                 # Restart
openclaw security audit --deep           # Security check
openclaw models status --probe --probe-provider anthropic  # Verify API access
```

## Architecture

- **Service Manager:** launchd (macOS)
- **Plist Location:** `~/Library/LaunchAgents/ai.openclaw.gateway.plist`
- **Config:** `~/.openclaw/openclaw.json`
- **Logs:** `~/.openclaw/logs/gateway.log` and `gateway.err.log`
- **Browser Data:** `~/.openclaw/browser/openclaw/user-data`
- **Runtime:** Node.js on `/opt/homebrew/bin/node`
- **Package:** `/opt/homebrew/lib/node_modules/openclaw/`
