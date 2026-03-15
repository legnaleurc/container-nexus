# Transmission proxy

Docker Compose project serving HTTP at `127.0.0.1:<TRANSMISSION_PORT>` on the host and
proxying Transmission at the configured upstream address. Transmission itself
runs remotely.
DNS, VPN connectivity, and certificates are managed outside this project.

Configure the host reverse proxy for HTTPS `torrent.example.com` to target
`http://127.0.0.1:<TRANSMISSION_PORT>`, preserving `Host: torrent.example.com`. Redirect HTTP
to HTTPS on the host. The container sets `X-Forwarded-Proto: https` because
the host reverse proxy terminates TLS. There is no HTTPS listener or HTTPS redirect in
this container.

## Setup

Nginx basic authentication is currently disabled.

Set `TRANSMISSION_PORT` and `TRANSMISSION_UPSTREAM` in your shell or copy
`.env.example` to `.env` and configure both before starting. Compose fails
if either value is unset or empty.

Validate and start from this directory:

```sh
docker compose config --quiet
docker compose run --rm --no-deps transmission nginx -t
docker compose up -d
```

Configuration:

- `TRANSMISSION_PORT`: localhost HTTP port (required; no default).
- `TRANSMISSION_UPSTREAM`: Transmission address as `host:port`, without a URL
  scheme or path (required; no default).

The official Nginx image renders `nginx.conf.template` at container startup.
After changing the template or environment, validate and recreate the container:

```sh
docker compose config --quiet
docker compose run --rm --no-deps transmission nginx -t
docker compose up -d --force-recreate
```

## Verification

Set `TRANSMISSION_PORT` in your shell to the configured port before running:

```sh
curl -i -H 'Host: torrent.example.com' "http://127.0.0.1:${TRANSMISSION_PORT}/"
curl -i -H 'Host: torrent.example.com' "http://127.0.0.1:${TRANSMISSION_PORT}/rpc"
```

The web UI should load; an RPC response of 409 requesting an
`X-Transmission-Session-Id` is normal.
The configuration retains the original `/`, `/rpc`, and `/upload` mappings.

Once access through the host reverse proxy works, remove the old Transmission site from the
server's Nginx configuration separately.
