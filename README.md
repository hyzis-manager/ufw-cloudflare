# UFW-Cloudflare

Automatically manage [Cloudflare IP ranges](https://www.cloudflare.com/ips/) in UFW (Uncomplicated Firewall). This script fetches the latest Cloudflare IPv4 and IPv6 addresses and creates firewall allow rules for HTTP/HTTPS traffic (ports 80 and 443 TCP), ensuring only Cloudflare-proxied requests reach your origin server.

Designed for **Ubuntu servers** (also compatible with Debian-based systems).

## Why?

When your domain is proxied through Cloudflare, all visitor traffic arrives from [Cloudflare IP addresses](https://developers.cloudflare.com/fundamentals/concepts/cloudflare-ip-addresses/) instead of individual visitor IPs. Your firewall must allow these ranges, otherwise legitimate traffic gets blocked. This script automates that process and keeps the rules up to date.

## Requirements

- **Ubuntu 20.04+** (or Debian-based system)
- `ufw` installed and enabled
- `wget`
- Root privileges (`sudo`)

## Quick Start

```bash
wget -O ufw.sh https://raw.githubusercontent.com/hyzis-manager/ufw-cloudflare/main/ufw.sh
chmod +x ufw.sh
sudo ./ufw.sh
```

On first run, the script will:

1. Fetch the current Cloudflare IPv4 and IPv6 ranges from `cloudflare.com/ips-v4` and `cloudflare.com/ips-v6`
2. Create UFW allow rules for each range on ports **80** and **443** (TCP)
3. Ask if you want to enable **Supervision** (automatic daily updates)

## Usage

```
sudo ./ufw.sh [options]
```

| Option | Short | Description |
|---|---|---|
| `--help` | `-h` | Show help message |
| `--purge` | `-p` | Remove all existing Cloudflare rules (tagged with `#cloudflare` comment) before adding new ones |
| `--no-new` | `-n` | Do not fetch IPs or add new rules (use with `--purge` to only remove rules) |
| `--supervision` | `-s` | Enable automatic daily updates via systemd timer |

## Examples

**First-time setup** — fetch and allow all Cloudflare IPs:

```bash
sudo ./ufw.sh
```

**Refresh rules** — remove outdated rules and add current ones:

```bash
sudo ./ufw.sh --purge
```

**Remove all Cloudflare rules** without adding new ones:

```bash
sudo ./ufw.sh --purge --no-new
```

**Enable daily auto-update** (Supervision):

```bash
sudo ./ufw.sh --supervision
```

## Supervision

The Supervision feature installs a **systemd timer** that runs the script once every 24 hours with `--purge`, ensuring your firewall rules always reflect the latest Cloudflare IP ranges.

When enabled, two systemd units are created:

- `ufw-cloudflare-supervision.service` — oneshot service that executes the script
- `ufw-cloudflare-supervision.timer` — timer that triggers the service daily (and 5 minutes after boot)

### Managing the timer

```bash
# Check timer status
systemctl status ufw-cloudflare-supervision.timer

# View next scheduled run
systemctl list-timers ufw-cloudflare-supervision.timer

# Disable auto-updates
sudo systemctl stop ufw-cloudflare-supervision.timer
sudo systemctl disable ufw-cloudflare-supervision.timer

# Manually trigger an update
sudo systemctl start ufw-cloudflare-supervision.service
```

## How It Works

1. Fetches IPv4 ranges from `https://www.cloudflare.com/ips-v4`
2. Fetches IPv6 ranges from `https://www.cloudflare.com/ips-v6`
3. For each CIDR range, runs: `ufw allow from <IP> to any port 80,443 proto tcp comment "cloudflare"`
4. When `--purge` is used, removes all existing UFW rules tagged with the `# cloudflare` comment before adding fresh ones
5. Validates IP formats and verifies the fetch was successful before applying changes

## Output Legend

During execution, the script displays progress indicators:

- **`+`** (green) — rule created
- **`-`** (red) — rule deleted
- **`.`** (gray) — rule skipped (already exists or invalid)

## Cloudflare IP Ranges

The script fetches IPs directly from Cloudflare's official endpoints. These ranges are also available via the [Cloudflare API](https://developers.cloudflare.com/api/resources/ips/methods/list/) at `https://api.cloudflare.com/client/v4/ips` (no authentication required). Cloudflare updates these ranges infrequently but recommends keeping your allowlist current.

## License

MIT
