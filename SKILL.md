---
name: vps-secure-bootstrap
description: Secure and harden a newly provisioned Ubuntu VPS using a Tailscale-first, zero-public-surface approach. Use when provisioning new servers, setting up cloud instances, preparing infrastructure for development tools, AI agents, automation services, or when a VPS must be locked down before deploying applications. Covers SSH hardening, firewall configuration, fail2ban, unattended security updates, and lockout prevention checks.
---

# VPS Secure Bootstrap

Secure a fresh Ubuntu 22.04+ VPS with a private-by-default architecture.

Default posture:
- Disable password authentication
- Disable root SSH login
- Avoid public SSH exposure
- Apply defense-in-depth controls
- Enable automatic security patching

# Safety Protocol (Mandatory)

Before modifying SSH or firewall settings:

1. Keep one active SSH session open.
2. Open a second test session before closing the first.
3. Do not disable root/password auth before confirming:
   - Non-root sudo user works
   - SSH key login works
4. Verify Tailscale connectivity before restricting public SSH.

Treat lockout prevention as a release-blocking requirement.

# Phase 1: Baseline Update

Run:

```bash
sudo apt update && sudo apt -y upgrade
sudo apt -y install curl ca-certificates gnupg
```

Proceed only after successful completion.

# Phase 2: Create Non-Root Sudo User (If Needed)

If logged in as root, run:

```bash
sudo adduser <ssh_user>
sudo usermod -aG sudo <ssh_user>
su - <ssh_user>
sudo whoami
```

Require `sudo whoami` to output `root`.

# Phase 3: Install Tailscale

Run:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
tailscale ip -4
```

From the local machine, verify:

```bash
ssh <ssh_user>@<tailscale_ip>
```

Do not proceed if this connection fails.

# Phase 4: Restrict Public Surface (Provider Firewall)

If using provider firewall controls (for example Hetzner), configure inbound rules to:
- Allow TCP 22 only from `tailscale_ip`
- Allow ICMP from any
- Deny all other inbound traffic

After applying rules, re-test SSH through Tailscale before closing existing sessions.

# Phase 5: Harden SSH

Edit `/etc/ssh/sshd_config` and ensure:

```text
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
ChallengeResponseAuthentication no
X11Forwarding no
```

Restart SSH:

```bash
sudo systemctl restart sshd
```

Validate:
- SSH through Tailscale still works
- `ssh root@<tailscale_ip>` fails

If validation fails, revert immediately.

# Phase 6: Configure UFW

Run:

```bash
sudo apt -y install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow in on tailscale0 to any port 22 proto tcp
sudo ufw enable
sudo ufw status verbose
```

Do not expose public SSH. For public web workloads, read `references/public-services.md`.

# Phase 7: Configure Fail2Ban

Run:

```bash
sudo apt-get install -y fail2ban
```

Create `/etc/fail2ban/jail.local`:

```ini
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 5

[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 3
bantime = 24h
```

Enable and validate:

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
sudo fail2ban-client status sshd
```

# Phase 8: Enable Unattended Security Updates

Run:

```bash
sudo apt-get install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
cat /etc/apt/apt.conf.d/20auto-upgrades
```

Expect:

```text
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

# Final Validation Checklist

Confirm all items before application deployment:
- SSH works through Tailscale IP
- Root SSH login is disabled
- Password authentication is disabled
- UFW default incoming policy is deny
- Fail2Ban `sshd` jail is active
- Unattended upgrades are enabled
- Public interface does not expose port 22

# Optional References

Load only when needed:
- `references/public-services.md`: Safe public HTTP/HTTPS exposure patterns
- `references/idempotency.md`: Safe re-runs on existing servers
- `references/recovery.md`: Provider-console lockout recovery steps
