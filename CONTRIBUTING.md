# Contributing to Headplane Coolify Deployment

Thank you for your interest in contributing! This guide outlines how to work with this project.

## Project Structure

```
.
├── docker-compose.yml      # Main deployment configuration
├── .env.example            # Environment variables template
├── .gitignore             # Git ignore patterns
├── README.md              # User documentation
├── CONTRIBUTING.md        # This file
├── headscale/
│   ├── config.yaml        # Headscale configuration
│   └── policy.hujson      # ACL policies
└── headplane/
    └── config.yaml        # Headplane UI configuration
```

## Before You Start

1. **Clone the repository**:
   ```bash
   git clone https://github.com/SajidK25/Headplane.git
   cd Headplane
   ```

2. **Set up environment**:
   ```bash
   cp .env.example .env
   # Edit .env with your actual values (DB passwords, OIDC secrets, etc.)
   ```

3. **Validate configuration**:
   ```bash
   docker-compose config --quiet
   ```

## Making Changes

### Configuration Changes

- **Headscale config**: Edit `headscale/config.yaml`
  - Database connections via environment variables
  - OIDC/Authentik integration settings
  - MagicDNS and ACL policy references
  - All sensitive values use `${ENV_VAR}` syntax

- **ACL Policies**: Edit `headscale/policy.hujson`
  - Define groups and tag ownership
  - Create access control rules
  - Configure SSH access
  - See [Tailscale ACL docs](https://tailscale.com/kb/1018/acls/)

- **Headplane UI**: Edit `headplane/config.yaml`
  - Web server settings
  - Headscale integration URLs
  - Docker socket configuration
  - Cookie and session settings

### Docker Compose Changes

When modifying `docker-compose.yml`:

1. **Preserve service dependencies**:
   - PostgreSQL must start before Headscale
   - Headscale must start before Headplane

2. **Never hardcode secrets**:
   - Use `${VAR_NAME}` for all sensitive values
   - Keep `.env.example` updated with new variables

3. **Maintain Traefik labels**:
   - All public services need Traefik routing labels
   - Include TLS configuration
   - Never use `expose:` directives (Traefik handles it)

4. **Healthchecks are required**:
   - PostgreSQL: `pg_isready`
   - Headscale: `headscale health`
   - Headplane: Node.js HTTP test (no wget/curl)

5. **Volume strategy**:
   - Data: Use Docker-managed volumes (`postgres_data`, etc.)
   - Configs: Mount read-only from repository (`./headscale/config.yaml:ro`)
   - Sockets: Mount Docker socket read-only

## Testing

### Local Validation

```bash
# Validate docker-compose syntax
docker-compose config --quiet

# Check environment variables
grep "^\$" docker-compose.yml
```

### Before Submitting PR

- [ ] Config files validate without errors
- [ ] All sensitive values use env vars
- [ ] `.env.example` is updated with new variables
- [ ] Traefik labels are present and correct
- [ ] Healthchecks are configured
- [ ] Docker volumes are named and consistent
- [ ] README reflects any changes
- [ ] No hardcoded credentials in any files

## Creating a Pull Request

1. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Commit your changes**:
   ```bash
   git add .
   git commit -m "Description of changes"
   ```

3. **Push and open PR**:
   ```bash
   git push origin feature/your-feature-name
   ```

4. **Fill out PR description**:
   - What changes were made
   - Why they were needed
   - Any new environment variables
   - Testing performed

## Documentation

### Update README.md if you add:
- New environment variables
- New Traefik routes
- Configuration options
- Troubleshooting tips

### Update .env.example when adding:
- New required environment variables
- New optional variables
- Changed variable names
- New defaults

## Code Standards

- **YAML files**: 2-space indentation
- **JSON files**: 2-space indentation (HuJSON comments allowed)
- **Comments**: Explain the "why", not just the "what"
- **Secrets**: Always use environment variables
- **Paths**: Use relative paths for mounted files

## Common Issues

### docker-compose config fails
```bash
# Check if .env file exists and has required vars
cat .env | grep "DB_PASSWORD\|OIDC_"
```

### Services won't start
```bash
# Check logs
docker-compose logs headscale
docker-compose logs headplane
docker-compose logs postgres
```

### Configuration changes not applied
```bash
# Rebuild without cache
docker-compose down
docker-compose up --build
```

## Performance Optimization Tips

1. **Database connections**:
   - Adjust `max_open_conns` and `max_idle_conns` in headscale config
   - Monitor query performance with `slow_threshold`

2. **DNS caching**:
   - MagicDNS responses are cached per-client
   - ACL changes require client reconnect

3. **WireGuard**:
   - Consider enabling `randomize_client_port` for better firewall traversal
   - Ensure UDP port 41641 is open (or randomized)

## Security Checklist

- [ ] All passwords use strong random values
- [ ] OIDC secrets are from trusted provider
- [ ] Cookie secrets are 32+ characters
- [ ] Database uses sslmode appropriate for network
- [ ] ACL policies restrict access correctly
- [ ] gRPC admin interface is on 127.0.0.1 only
- [ ] Traefik TLS is enabled for all public services
- [ ] No Docker socket exposure outside container

## Getting Help

- **Headscale docs**: https://headscale.net/
- **Tailscale KB**: https://tailscale.com/kb/
- **Authentik docs**: https://goauthentik.io/
- **Coolify docs**: https://coolify.io/

## License

By contributing, you agree that your contributions will be licensed under the same license as the project.
