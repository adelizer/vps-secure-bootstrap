# VPS Secure Bootstrap
Secure and harden a fresh Ubuntu VPS (22.04+) with a private-by-default, Tailscale-first setup.

## What this does

- Disables SSH password authentication
- Disables root SSH login
- Keeps SSH private over Tailscale
- Adds firewall and brute-force protections
- Enables unattended security updates

## Quick flow

1. Update the system and install base packages.
2. Create/use a non-root sudo user.
3. Install Tailscale and verify SSH over Tailnet.
4. Restrict public inbound access at provider firewall.
5. Harden SSH (`PermitRootLogin no`, `PasswordAuthentication no`).
6. Configure UFW to allow SSH on `tailscale0` only.
7. Enable Fail2Ban for SSH.
8. Enable unattended security upgrades.

## Safety first

Before changing SSH or firewall settings:

- Keep one active SSH session open.
- Open a second test session before closing the first.
- Verify key-based login with non-root sudo user.
- Verify Tailscale SSH works before restricting public SSH.

## Detailed instructions

Use `SKILL.md` for the full, step-by-step procedure and the files in `references/` for special cases:

- `references/public-services.md`
- `references/idempotency.md`
- `references/recovery.md`
