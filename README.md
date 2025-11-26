# Traefik to Pi-hole DNS Sync

Automatically discovers services exposed by Traefik and creates corresponding DNS records in Pi-hole.

## Features

- 🔄 Automatically syncs Traefik-exposed services to Pi-hole DNS
- ⏰ Runs on a configurable schedule (default: every 5 minutes)
- 🔍 Dry-run mode to preview changes
- 🐳 Runs as a Docker container
- 📝 Comprehensive logging

## How It Works

1. Queries the Traefik API to discover all HTTP routers
2. Extracts hostnames from router rules (e.g., `Host(\`example.com\`)`)
3. Connects to Pi-hole and checks existing DNS records
4. Adds missing DNS records pointing to your Traefik host IP

## Quick Start

### 1. Configure Pi-hole

Enable API write access in Pi-hole:

1. Log in to your Pi-hole web interface
2. Navigate to **Settings** > **API / Web interface**
3. Find the setting **`webserver.api.app_sudo`**
4. Enable it: *"Should application password API sessions be allowed to modify config settings?"*
5. Click **Save**

> [!IMPORTANT]
> This setting is required for the sync tool to add DNS records. Without it, API sessions will be read-only.

### 2. Get Your Pi-hole Password

You can retrieve your App Password from the Pi-hole web interface:

1. Log in to your Pi-hole web interface.
2. Navigate to **Settings** > **API / Web interface**.
3. Retrieve your **App Password** (or API Token) from this page.

### 3. Create Environment File

```bash
cp .env.example .env
```

Edit `.env` and configure:

```bash
# Traefik API URL (internal Docker network)
TRAEFIK_API_URL=http://traefik:8080/api/http/routers

# Pi-hole URL (accessible from Docker container)
PIHOLE_URL=https://pihole.example.com

# Pi-hole web admin password
PIHOLE_PASSWORD=your-pihole-password

# IP address that DNS records should point to (your Traefik host)
TRAEFIK_HOST_IP=192.168.1.100

# How often to sync (cron format or @every syntax)
SYNC_INTERVAL=@every 5m

# Log level: info or debug
LOG_LEVEL=info
```

### 4. Run

Using Docker Compose:

```bash
docker compose up -d
```

Or run manually using the pre-built image:

```bash
docker run -d \
  --name traefik-pihole-dns-sync \
  --env-file .env \
  ghcr.io/rpressiani/traefik-pihole-dns-sync:latest
```

### 5. Test with Dry-run

Before making actual changes:

```bash
docker run --rm --env-file .env ghcr.io/rpressiani/traefik-pihole-dns-sync:latest /app/sync --dry-run
```

Or run once without scheduling:

```bash
docker run --rm --env-file .env ghcr.io/rpressiani/traefik-pihole-dns-sync:latest /app/sync --once
```

## Testing Tools

Test the Traefik API directly:

```bash
cd tools
go build -o test-api test-api.go
./test-api --rules
```

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `TRAEFIK_API_URL` | `http://traefik:8080/api/http/routers` | Traefik API endpoint |
| `PIHOLE_URL` | `https://pihole.example.com` | Pi-hole web interface URL |
| `PIHOLE_PASSWORD` | (required) | Pi-hole admin password |
| `TRAEFIK_HOST_IP` | (required) | IP address for DNS A records |
| `SYNC_INTERVAL` | `@every 5m` | Sync frequency (cron or @every) |
| `LOG_LEVEL` | `info` | Log verbosity: `info` or `debug` |

### Cron Interval Syntax

You can use standard cron expressions or the `@every` syntax:

- `@every 5m` - Every 5 minutes
- `@every 1h` - Every hour
- `*/10 * * * *` - Every 10 minutes (standard cron)
- `0 */2 * * *` - Every 2 hours (standard cron)

## Command-Line Flags

- `--once` - Run sync once and exit (useful for testing)
- `--dry-run` - Show what would be synced without making changes

## Monitoring

View logs:

```bash
docker compose logs -f traefik-pihole-dns-sync
```

Check sync status:

```bash
docker compose exec traefik-pihole-dns-sync /app/sync --once --dry-run
```

## Development

To build and test changes locally without pushing to GitHub:

1. Edit `docker-compose.yml` and uncomment `build: .` (comment out `image: ...`)
2. Build and run:
   ```bash
   docker compose up -d --build
   ```

### Testing with Flags

To test with `--dry-run` or `--once` flags:

```bash
# Build the image
docker compose build

# Run once with dry-run (no changes made)
docker compose run --rm traefik-pihole-dns-sync /app/sync --dry-run --once

# Run once (makes actual changes)
docker compose run --rm traefik-pihole-dns-sync /app/sync --once
```

## Troubleshooting

### "Failed to get Traefik routers"

- Ensure Traefik API is enabled and accessible
- Check that the container is on the same network as Traefik
- Verify `TRAEFIK_API_URL` is correct

### "Failed to authenticate with Pi-hole"

- Verify `PIHOLE_PASSWORD` is correct
- Ensure Pi-hole is accessible from the container
- Check Pi-hole logs for authentication errors

### "No hostnames found to sync"

- Verify your Traefik routers have `Host()` rules
- Check that routers are enabled
- Run with `LOG_LEVEL=debug` for more details

## Security Notes

> [!NOTE]
> This tool uses Pi-hole v6 session-based authentication. It requires your Pi-hole App Password to generate a session ID. The password is only used to obtain the session and is not stored or logged.

## TODO

- [ ] Add environment variables for run modes (e.g., `RUN_MODE=dry-run` or `RUN_MODE=once`)
- [ ] Add support for removing stale DNS records
- [ ] Add Prometheus metrics
- [ ] Support for CNAME records instead of A records
- [ ] Webhook notifications on sync errors

## License

MIT
