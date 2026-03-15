# DFD

Copy `.env.example` to `.env` and configure:

- `DULD_API_URL`: the DULD API HTTP origin using its reachable VPN IP and port.
  The example IP is a placeholder.

The container host must already have a working route to the IP through the VPN.
DULD must accept the IP address in the HTTP Host header.

For an existing deployment, set `DULD_API_URL` to the HTTP origin and remove
`DULD_API_HOST`, `DULD_API_VPN_IP`, and `DULD_API_AUTHORIZATION` from `.env`.

From this directory, apply configuration changes by recreating the container:

```sh
docker compose up -d --force-recreate
```
