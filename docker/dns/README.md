# Split-Horizon DNS Server

This Docker Compose project sets up dnsmasq for local DNS resolution with split-horizon capability.

## Purpose

- At home: Resolves `nas.example.com` to local IP (192.0.2.100)
- On VPN: VPS DNS resolves `nas.example.com` to VPN IP (198.51.100.2)

The IP addresses below are documentation placeholders. Replace them with your actual LAN and VPN addresses before deployment.

## Setup Instructions

### 1. Edit Configuration

Copy the example environment file and dnsmasq config if needed:
```bash
cp .env.example .env
cp -r dnsmasq.d.example dnsmasq.d
```

Then edit `.env`:
```bash
DNS_CONFIG_PATH=/path/to/container-nexus/docker/dns/dnsmasq.d
DNS_HOST_BIND_ADDRESS=192.0.2.100
```

The real `.env` file and `dnsmasq.d/` folder are local-only; commit changes to `.env.example` and `dnsmasq.d.example/` instead.

`DNS_HOST_BIND_ADDRESS` controls the Docker host-side port binding, so DNS is not published on every host interface.
No listen settings are needed: the image's base config sets `interface=*`, which keeps dnsmasq from enabling `local-service` (that would drop LAN queries, since Docker keeps the client's source IP).

`DNS_CONFIG_PATH` is the host folder mounted read-only at `/etc/dnsmasq.d`; every `*.conf` in it is loaded. `dnsmasq.d.example/` in this repo is a working sample; the local `dnsmasq.d/` copy is git-ignored.
Set upstream DNS with `server=` lines in your config. Without any, the image falls back to its built-in `DNS1`/`DNS2` defaults (1.0.0.1, 1.1.1.1).
Do not mount `/etc/dnsmasq.conf` directly: the image only generates its base config when that file is absent.

### 2. Deploy with Docker Compose

```bash
# Start the DNS server
docker-compose up -d

# Check if it's running
docker-compose ps

# View logs
docker-compose logs -f
```

### 3. Configure Your Router

Set the DHCP DNS server to your Docker host’s LAN IP address (192.0.2.100) so all home devices use this DNS server.

If your router doesn't support custom DHCP DNS settings, configure each device manually.

### 4. Test

From a device on your home network:
```bash
nslookup nas.example.com
# Should return: 192.0.2.100

nslookup google.com
# Should return: public Google IP (verifying upstream DNS works)
```

## Management Commands

```bash
# Start
docker-compose up -d

# Stop
docker-compose down

# Restart (after config changes)
docker-compose restart

# View logs
docker-compose logs -f dnsmasq

# Update image
docker-compose pull
docker-compose up -d
```

## Troubleshooting

### Port 53 already in use
Check for another DNS service using port 53 on the configured bind address (for example, Synology’s DNS Server package on DSM). Stop or reconfigure the conflicting service, or bind this project to another available host IP address. DHCP clients normally expect DNS on port 53.

### DNS not resolving
1. Check if container is running: `docker-compose ps`
2. Check logs: `docker-compose logs dnsmasq`
3. Verify firewall allows UDP/TCP port 53
4. Ensure devices are pointing to the correct DNS server

## VPS Setup (for remote access)

On your VPS, set up similar dnsmasq configuration:
```
address=/nas.example.com/198.51.100.2
```

And configure OpenVPN to push the VPS as DNS server to clients.
