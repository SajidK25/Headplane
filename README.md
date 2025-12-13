# Headscale + Headplane on Coolify

Production-ready Tailscale control plane deployment with Authentik SSO and Traefik TLS.

## Quick Setup

| Step | Action |
|------|--------|
| 1 | Import Git repo: `https://github.com/SajidK25/Headplane.git` |
| 2 | Set environment variables (see below) |
| 3 | Map domains via `SERVICE_FQDN` variables |
| 4 | Deploy and access via HTTPS |

## Environment Variables

| Variable | Purpose | Example |
|----------|---------|---------|
| `DB_PASSWORD` | Postgres password | Strong random string |
| `OIDC_CLIENT_ID` | Authentik OAuth2 client ID | `headscale` |
| `OIDC_CLIENT_SECRET` | Authentik OAuth2 secret | From Authentik provider |
| `OIDC_ISSUER` | Authentik OIDC endpoint | `https://auth.domain.com/application/o/headscale/` |
| `HEADPLANE_COOKIE_SECRET` | Session encryption key | `openssl rand -base64 32` |
| `SERVICE_FQDN` | Headscale domain | `headscale.example.com` |
| `SERVICE_FQDN_HEADPLANE` | Headplane domain | `headplane.example.com` |

**Optional:**
- `TIMEZONE` (default: `America/New_York`)
- `LOG_LEVEL` (default: `info`)

## Traefik Configuration

Coolify automatically handles TLS and routing. Services:

| Service | Port | Domain | Routing |
|---------|------|--------|---------|
| Headscale | 8080 | `${SERVICE_FQDN}` | HTTP→HTTPS + TLS |
| Headplane | 3000 | `${SERVICE_FQDN_HEADPLANE}` | HTTP→HTTPS + TLS |
| Postgres | 5432 | Internal | Not exposed |

## Architecture

```
    ┌──────────────────────┐
    │ Coolify + Traefik    │
    │ (TLS, Routing)       │
    └─────────┬────────────┘
        ┌─────┴────────┐
        │              │
    Headscale     Headplane
    (8080)        (3000)
        │              │
        └─────┬────────┘
              │
         PostgreSQL
```

## File Structure

```
.
├── docker-compose.yml
├── headscale/
│   ├── config.yaml
│   └── policy.hujson
└── headplane/
    └── config.yaml
```

## Volumes

Docker-managed (Coolify persists):

| Volume | Purpose |
|--------|---------|
| `postgres_data` | Database |
| `headscale_data` | VPN state & keys |
| `headplane_data` | Web UI cache |

Config files: read-only mounts from repo.

## Authentik Setup

### 1. Create Provider

Admin → Applications → Providers → New OpenID Connect:
- **Name**: `Headscale`
- **Client ID**: `headscale`
- **Redirect URI**: `https://${SERVICE_FQDN}/oidc/callback`
- **Scopes**: `openid`, `profile`, `email`

### 2. Create Application

Admin → Applications → Applications:
- **Name**: `Headscale`
- **Provider**: Above
- **Flow**: Default

### 3. Set Coolify Env Vars

```
OIDC_CLIENT_ID=headscale
OIDC_CLIENT_SECRET=<from-provider>
OIDC_ISSUER=https://auth.domain.com/application/o/headscale/
```

## Connect Clients

```bash
tailscale up --login-server https://${SERVICE_FQDN}
# Opens browser for Authentik SSO
# Node registered as: nodename.user.tail.megabyte.space
```

## Configuration Files

### `headscale/config.yaml`
- Database: PostgreSQL (env vars)
- OIDC: Authentik integration
- MagicDNS: Base domain configured
- ACL: From `policy.hujson`

### `headscale/policy.hujson`
- **Groups**: `admins`, `dev`
- **Tags**: `infra`, `apps`, `db`, `iot`, `router`, `exit`
- **Rules**: Hierarchical access control

### `headplane/config.yaml`
- Server: 0.0.0.0:3000
- Headscale: Internal HTTP
- Docker: Integration enabled

## API Key Auth (Optional)

```bash
docker exec headscale headscale apikey create --expiration 999d
# Use output as Headplane password
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Won't start | Check logs: `docker logs <container>` |
| OIDC fails | Verify redirect URI in Authentik matches `SERVICE_FQDN` |
| Can't connect | Check firewall: ports 443 (HTTPS), UDP (WireGuard) |
| DNS fails | Run: `tailscale set --accept-dns=true` |

## Security

- ✅ TLS via Traefik (Let's Encrypt)
- ✅ OIDC (no hardcoded credentials)
- ✅ gRPC on 127.0.0.1 only
- ✅ Configs read-only mounted
- ✅ Strong DB password required

## Resources

- [Headscale Docs](https://headscale.net/)
- [Tailscale KB](https://tailscale.com/kb/)
- [Authentik Docs](https://goauthentik.io/)
- [Coolify Docs](https://coolify.io/)
