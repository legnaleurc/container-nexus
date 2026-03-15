# HaH

Runs the CLI JAR using rootless Podman Compose and Eclipse Temurin Java 25. The JAR is mounted read-only;
only the configured storage folder and an ephemeral container `/tmp` are writable.
Uses Podman’s default rootless user namespace: container root maps to the
account running Podman on the host. Run Podman as the storage owner, without sudo.
No explicit user namespace is configured, allowing pod-based systemd integration.

## Setup

1. Copy `.env.example` to `.env` and set the host paths, bind address,
   and client port. Paths refer to the Podman host. Match `HAH_PORT`
   to the port registered for your H@H client; forward that TCP port to the host.
2. Place your trusted `HaH.jar` at `HAH_JAR_PATH`. Create
   `HAH_STORAGE_PATH` and give the account running Podman write access. Keep
   the JAR outside the writable storage folder. Do not make storage world-writable.
3. Validate the resolved configuration:

   ```sh
   podman compose config --quiet
   ```

4. Run initial registration interactively:

   ```sh
   podman compose run --rm --service-ports hah
   ```

   Enter the client credentials when prompted. Once registration and startup
   succeed, stop with Ctrl+C, then run the persistent service:

   ```sh
   podman compose up -d
   podman compose logs -f hah
   ```

Do not run the initial registration container and persistent service simultaneously.
Client credentials persist in `storage/data/client_login`; protect the storage
folder and its backups. To migrate an existing installation, stop it first and
copy its `data`, `cache`, `download`, `log`, and `tmp` directories into the storage
folder, preserving ownership appropriate for the account running Podman.

The image major version is pinned to 25. When managed by systemd, update with
`podman-compose -p hah pull` followed by
`systemctl --user restart podman-compose@hah.service` after installing the
drop-in below. Java 25 compatibility with this
older client has not been runtime-tested here. Verify registration, connectivity,
cache operations, and downloads on the host.

Logging uses journald to match the other projects in this repository. H@H also
writes its application logs under `storage/log`.

## Podman requirements

`podman compose` delegates to an external Compose provider. Install a current
`podman-compose` and select it consistently when running these commands:

```sh
export PODMAN_COMPOSE_PROVIDER=podman-compose
```

The rootless account needs subordinate UID/GID mappings configured on the host.
Use a client port of 1024 or greater with the default rootless host settings.
Keep the JAR readable and storage writable by the account running Podman;
access granted only through supplementary groups may need additional setup.

On SELinux-enforcing hosts, add `selinux: Z` under each mount’s `bind` settings
for these dedicated H@H paths. Debian normally does not need this relabeling.

`restart: unless-stopped` handles process restarts; arrange startup after a host
reboot separately using the host’s Podman/systemd setup. Rootless services that
must survive logout usually require user lingering. This project does not change
host services or lingering settings.

Reference: [Podman Compose](https://docs.podman.io/en/latest/markdown/podman-compose.1.html)
and [Podman run user namespaces and mounts](https://docs.podman.io/en/latest/markdown/podman-run.1.html).

## Incoming source addresses

The service uses `slirp4netns:port_handler=slirp4netns` to preserve incoming
source IPs. HaH checks API server source addresses during startup; the default
rootless bridge forwarding rewrites those addresses and can cause
`FAIL_CONNECT_TEST` with a private source address in the rejection log.

This configuration targets Podman 5.4.2 and podman-compose 1.3.0, whose default
pod arguments are `--infra=false --share=`. Keep these defaults: the service
has its own network namespace. Do not combine this network mode with service
`networks` entries or enable shared pod networking.

After changing the network mode, recreate the container rather than merely
restarting the existing container. On the host, from this project directory:

```sh
systemctl --user stop podman-compose@hah.service
podman-compose -p hah down
systemctl --user reset-failed podman-compose@hah.service
systemctl --user start podman-compose@hah.service
podman-compose -p hah logs -f hah
```

`down` removes the project containers and pod; the bind-mounted storage remains.

## Systemd cleanup override

Install the project’s drop-in on the host as the account running Podman:

```sh
mkdir -p ~/.config/systemd/user/podman-compose@hah.service.d
cp systemd/override.conf ~/.config/systemd/user/podman-compose@hah.service.d/override.conf
systemctl --user daemon-reload
systemctl --user cat podman-compose@hah.service
```

The drop-in assumes the project is at `~/local/src/container-nexus/docker/hah` and
podman-compose is at `/usr/bin/podman-compose`. Adjust those paths if needed.
It removes the stopped project containers and pod after each service stop,
so the next start creates them from the current configuration and local image.
Bind-mounted data is preserved; this does not pull images automatically.

podman-compose 1.3.0 already checks the Compose configuration hash, but does
not check whether the image behind the same tag has changed. Pull first, then
restart the systemd service to deploy an image update.
