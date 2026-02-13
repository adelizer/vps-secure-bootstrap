# Lockout Recovery

Use this only after access loss.

## Recovery Procedure

1. Open provider console access.
2. Disable UFW temporarily:

```bash
sudo ufw disable
```

3. If required, temporarily relax `/etc/ssh/sshd_config`.
4. Restart SSH:

```bash
sudo systemctl restart sshd
```

5. Restore key-based non-root access.
6. Reapply hardening controls in a safe sequence.
