# OpenClaw on macOS M1 — Setup Guide

A clean installation guide for setting up OpenClaw and Claude Code on a fresh MacBook Pro M1.

> **Note:** The [innerclaw](https://github.com/xclaw-labs/innerclaw) hardening scripts are designed for Ubuntu servers and do not run on macOS. This guide covers the macOS-native equivalent steps.

---

## Prerequisites

- MacBook Pro M1 (clean install of macOS)
- Apple ID for the machine
- Agent GitHub account (e.g. `aidax-axel`) with a fine-grained personal access token
- Agent email address (e.g. `axel@yourcompany.com`)

---

## Step 1 — Base tools

```bash
# Xcode CLI tools (required for git, compilers)
xcode-select --install

# Homebrew (installs to /opt/homebrew on M1)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Add Homebrew to PATH
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"

# Node.js via nvm
brew install nvm
echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.zprofile
echo '[ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && . "/opt/homebrew/opt/nvm/nvm.sh"' >> ~/.zprofile
source ~/.zprofile

nvm install --lts
nvm use --lts
node --version  # should print v22.x or later
```

---

## Step 2 — Claude Code

```bash
npm install -g @anthropic-ai/claude-code
claude --version

# Log in with the agent's account (not your personal account)
claude login
# Opens browser → authenticate with the agent's credentials
```

---

## Step 3 — GitHub CLI and agent identity

```bash
brew install gh

# Authenticate with the agent's GitHub account
gh auth login
# Select: GitHub.com → HTTPS → Paste an authentication token
# Paste the agent's fine-grained personal access token

# Set git identity
git config --global user.name "Axel (aidax)"
git config --global user.email "axel@yourcompany.com"
git config --global init.defaultBranch main

# Verify
gh auth status
```

---

## Step 4 — OpenClaw

```bash
npm install -g openclaw
openclaw --version

# Initialize OpenClaw
openclaw init
```

To copy config from your primary machine:

```bash
# On your primary machine, copy the config:
scp ~/.openclaw/config.yaml new-mac-hostname:~/.openclaw/config.yaml

# Or reconfigure manually on the new machine
```

---

## Step 5 — Agent workspace

```bash
mkdir -p ~/workspace
cd ~/workspace

# Clone the repos this agent will work on
gh repo clone aidax-labs/aidax-api
gh repo clone aidax-labs/aidax-app
# etc.

# Place the agent's CLAUDE.md in the workspace root
# (get it from the aidax-labs/agentic-company-playbook repo)
cp agentic-company-playbook/agent-configs/axel-CLAUDE.md ~/workspace/CLAUDE.md
```

---

## Step 6 — macOS hardening (innerclaw equivalent)

These are the macOS-native controls that approximate innerclaw's Tier 1 security posture.

### Verify SIP is active
```bash
csrutil status
# Expected: "System Integrity Protection status: enabled"
# Never disable SIP on an agent machine
```

### Enable the macOS firewall
```bash
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setstealthmode on

# Verify
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
```

### Disable SSH (unless needed)
```bash
# Check current status
sudo systemsetup -getremotelogin

# Disable if not needed
sudo systemsetup -setremotelogin off
```

### Verify FileVault (disk encryption)
```bash
fdesetup status
# If not enabled:
sudo fdesetup enable
# Follow prompts — save the recovery key in your password manager
```

### Automatic updates
```bash
# Enable automatic security updates via System Settings:
# System Settings → General → Software Update → Automatic Updates → ON
# Enable: Security Responses and system files
```

### Disable Bluetooth and AirDrop if not needed
```bash
# Via System Settings → Bluetooth → Turn Off
# Via Finder → AirDrop → Allow discovery: No One
```

---

## Step 7 — Auto-start with launchd (optional)

To have OpenClaw start automatically when the machine boots:

```bash
# Create the launchd plist
cat > ~/Library/LaunchAgents/com.aidax.openclaw.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.aidax.openclaw</string>
  <key>ProgramArguments</key>
  <array>
    <string>/opt/homebrew/bin/node</string>
    <string>/opt/homebrew/bin/openclaw</string>
    <string>start</string>
  </array>
  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/>
  <key>StandardOutPath</key>
  <string>/tmp/openclaw.log</string>
  <key>StandardErrorPath</key>
  <string>/tmp/openclaw.err</string>
</dict>
</plist>
EOF

# Load it
launchctl load ~/Library/LaunchAgents/com.aidax.openclaw.plist

# Verify it's running
launchctl list | grep openclaw
```

To stop/unload:
```bash
launchctl unload ~/Library/LaunchAgents/com.aidax.openclaw.plist
```

---

## innerclaw vs macOS — control mapping

| innerclaw (Ubuntu) | macOS equivalent | Notes |
|---|---|---|
| UFW deny-all | macOS Firewall + stealth mode | Steps 6 above |
| fail2ban | Not needed | macOS SSH off by default |
| AppArmor sandbox | SIP (always on M1) | Cannot be replicated exactly |
| iptables egress allowlist | [Little Snitch](https://www.obdev.at/littlesnitch) (paid) | Best egress control on macOS |
| systemd service | launchd plist | Step 7 above |
| FileVault | FileVault (native) | Same concept, different name |
| Vaultwarden | 1Password CLI | `brew install 1password-cli` |
| Audit log | macOS Unified Log | `log stream --predicate 'process == "openclaw"'` |

---

## Verification checklist

- [ ] `csrutil status` → enabled
- [ ] `fdesetup status` → FileVault is On
- [ ] `sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate` → enabled
- [ ] `sudo systemsetup -getremotelogin` → Remote Login: Off
- [ ] `claude --version` → installed
- [ ] `gh auth status` → logged in as agent account
- [ ] `git config --global user.email` → agent email
- [ ] `openclaw --version` → installed
- [ ] `launchctl list | grep openclaw` → running (if auto-start configured)
