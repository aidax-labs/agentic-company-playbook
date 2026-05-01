# OpenClaw on Linux Mint 22.1 — Setup Guide with innerclaw hardening

Target hardware: MacBook Air Intel i5, 4GB RAM, Linux Mint 22.1 (Ubuntu 24.04 base)

---

## RAM budget (4GB)

| Component | RAM |
|---|---|
| Linux Mint desktop (idle) | ~1.2 GB |
| OpenClaw + Claude Code | ~500 MB |
| Vaultwarden (Docker) | ~100 MB |
| OS headroom | ~200 MB |
| **Available for agent work** | **~2 GB** |

**Recommendation:** Switch to Xfce session or disable the display manager during agent operation to free ~700 MB:

```bash
# Disable display manager (run headless, SSH in to manage)
sudo systemctl disable lightdm --now

# Re-enable desktop when needed
sudo systemctl enable lightdm --now && sudo systemctl start lightdm
```

**Tier 3 with local model (Ollama/Qwen) is not viable on 4GB.** Minimum 8GB needed for a 7B model.

---

## Part 1 — Base system prep

```bash
# Update everything first
sudo apt update && sudo apt upgrade -y

# Required packages (innerclaw script 10 covers these, but good to verify)
sudo apt install -y curl git wget unzip build-essential \
  ca-certificates gnupg lsb-release

# Verify AppArmor is active (should be enabled by default on Mint 22.1)
sudo aa-status
# Expected: apparmor module is loaded

# Verify systemd
systemctl --version
```

---

## Part 2 — Node.js

```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc

# Install LTS
nvm install --lts
nvm use --lts
node --version   # v22.x or later
npm --version
```

---

## Part 3 — Claude Code

```bash
npm install -g @anthropic-ai/claude-code
claude --version

# Log in with the agent's account
claude login
```

---

## Part 4 — GitHub CLI and agent identity

```bash
# Install gh
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \
  | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] \
  https://cli.github.com/packages stable main" \
  | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update && sudo apt install -y gh

# Authenticate with the agent's GitHub account
gh auth login
# GitHub.com → HTTPS → Paste an authentication token

# Set git identity
git config --global user.name "Leo (aidax)"
git config --global user.email "leo@yourcompany.com"
git config --global init.defaultBranch main
```

---

## Part 5 — OpenClaw

```bash
npm install -g openclaw
openclaw --version
openclaw init
```

---

## Part 6 — innerclaw hardening (Tier 1)

Clone the innerclaw repo and run scripts in order.

```bash
git clone https://github.com/xclaw-labs/innerclaw.git
cd innerclaw

# Copy and edit config
cp config.env.example config.env
nano config.env
```

Key values to set in `config.env`:
```bash
ADMIN_USER=your-username        # your Linux user
SSH_PORT=13579                  # non-standard SSH port
WG_PORT=51820                   # WireGuard port
WG_SERVER_IP=10.100.0.1        # VPN server IP
OPENCLAW_USER=openclaw          # service user for OpenClaw
```

Run Tier 1 scripts in order:
```bash
# 1. Host baseline: updates, base packages, openclaw user, fail2ban
sudo bash scripts/10_baseline_host.sh --config ./config.env

# 2. Network: SSH hardening (key-only, custom port), UFW deny-all
sudo bash scripts/20_network_access.sh --config ./config.env

# 3. WireGuard VPN (prints client config at the end — save it)
sudo bash scripts/25_wireguard_vpn.sh --config ./config.env

# 4. OpenClaw directories, permissions, systemd service
sudo bash scripts/30_openclaw_baseline.sh --config ./config.env

# 5. Install OpenClaw on host
sudo bash scripts/50_install_openclaw_host.sh --config ./config.env

# 6. Harden OpenClaw config (auth, logging, limits)
sudo bash scripts/51_openclaw_config_hardening.sh --config ./config.env

# 7. Kill switch (emergency stop)
sudo bash scripts/52_kill_switch.sh --config ./config.env

# 8. Skill pre-install validator (IOC + typosquat + AI analysis)
sudo bash scripts/53_skill_preinstall_validator.sh --config ./config.env

# 9. Validate everything
sudo bash scripts/40_validate.sh --config ./config.env
```

---

## Part 7 — innerclaw hardening (Tier 2)

Only if Tier 1 passes validation cleanly.

```bash
# Disable unnecessary system services (frees RAM)
sudo bash scripts/12_minimal_services.sh --config ./config.env

# Admin privilege hardening
sudo bash scripts/15_admin_hardening.sh --config ./config.env

# Kernel hardening: sysctl, ASLR, IPv6 off, dmesg_restrict
sudo bash scripts/22_kernel_hardening.sh --config ./config.env

# Egress allowlist (anti-exfiltration — key for agent security)
sudo bash scripts/35_egress_control.sh --config ./config.env

# Append-only audit log
sudo bash scripts/45_audit_log.sh --config ./config.env

# AppArmor profile for OpenClaw process
sudo bash scripts/55_apparmor_profile.sh --config ./config.env

# Daily backup of OpenClaw data
sudo bash scripts/60_openclaw_backup.sh --config ./config.env

# Vaultwarden (secrets manager, VPN-only access)
# Note: requires Docker — see below for 4GB RAM impact
sudo bash scripts/70_deploy_edge_stack.sh --config ./config.env
```

---

## Docker on 4GB RAM

Vaultwarden in Docker uses ~100MB. Docker itself adds ~150MB overhead. On 4GB this is acceptable but monitor memory:

```bash
# Install Docker
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
newgrp docker

# After running script 70, verify Vaultwarden is up
docker ps | grep vaultwarden

# Monitor RAM usage
free -h
# If memory pressure is high, skip Vaultwarden and use 1Password CLI instead:
# sudo apt install 1password-cli
```

---

## Linux Mint specific notes

Linux Mint 22.1 is based on Ubuntu 24.04. A few differences from vanilla Ubuntu Server:

| Area | Mint 22.1 | innerclaw assumption | Action needed |
|---|---|---|---|
| Desktop environment | Cinnamon (default) | Server / no DE | Disable or switch to Xfce |
| Snap packages | Disabled by default | Not relevant | No action |
| UFW | Installed, may be inactive | Managed by script 20 | Script handles it |
| AppArmor | Active | Active | No action |
| systemd | Active | Active | No action |
| Firewall GUI | `gufw` installed | Not relevant | No action |

---

## Agent workspace setup

```bash
# Create workspace directory for this agent
mkdir -p ~/workspace
cd ~/workspace

# Clone repos this agent will work on
gh repo clone aidax-labs/aidax-api
# etc.

# Place agent's CLAUDE.md
# (from aidax-labs/agentic-company-playbook/agent-configs/)
```

---

## Verification checklist

- [ ] `sudo aa-status` → apparmor module is loaded
- [ ] `sudo ufw status` → active, deny-all incoming
- [ ] `sudo systemctl status fail2ban` → active (running)
- [ ] `sudo systemctl status openclaw` → active (running)
- [ ] `sudo wg show` → WireGuard interface up
- [ ] `claude --version` → installed
- [ ] `gh auth status` → logged in as agent account
- [ ] `git config --global user.email` → agent email
- [ ] `docker ps | grep vaultwarden` → running (Tier 2 only)
- [ ] `free -h` → available memory > 1GB at idle
- [ ] `sudo bash scripts/40_validate.sh --config ./config.env` → 0 FAILs

---

## Memory optimization tips for 4GB

```bash
# Check what's using RAM
ps aux --sort=-%mem | head -20

# Disable swap if it's on an SSD (reduces wear)
# Or increase swappiness to reduce swap usage:
sudo sysctl vm.swappiness=10
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf

# Free page cache if needed
sudo sync && sudo sh -c 'echo 3 > /proc/sys/vm/drop_caches'
```
