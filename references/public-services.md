# Public Services Exposure

Use this only when public web access is required.

## Allow HTTP and HTTPS

Run:

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Keep SSH restricted to `tailscale0`.

## Use a Reverse Proxy

Prefer one of:
- Caddy
- Nginx
- Traefik

Terminate TLS at the proxy.
Bind internal services to `127.0.0.1` or `tailscale0`.

## Never Do These

- Expose databases publicly
- Expose internal dashboards publicly
- Disable firewall rules for debugging
