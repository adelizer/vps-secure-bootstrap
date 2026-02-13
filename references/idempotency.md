# Idempotent Re-Runs

Before re-running hardening steps, verify current state.

## Check Existing User

```bash
id <ssh_user>
```

## Check Tailscale

```bash
tailscale status
```

## Check UFW

```bash
sudo ufw status verbose
```

## Check Fail2Ban

```bash
sudo systemctl status fail2ban
```

## Re-Run Rules

- Do not recreate existing users
- Do not duplicate firewall rules
- Overwrite `jail.local` intentionally when updating policy
