# Drive (alpha)

Internal test deployment of `wcpan.drive.server`, built from the wcpan.drive
workspace instead of PyPI. It runs alongside the stable `drive` project.

## Setup

Copy `.env.example` to `.env` and configure the source, host paths and
listening address. `DRIVE_SOURCE` is a git URL (pin a branch, tag or commit
with `#ref`) or an absolute path to a local checkout. Create the
`DRIVE_DATA_PATH` and `DRIVE_WATCH_PATH` directories and the config file
before starting. Give the container user write access to `DRIVE_DATA_PATH` and
read access to the configuration file and watch directory. All bind sources
must already exist; Compose will fail instead of creating missing directories.

The container uses the host timezone through a read-only `/etc/localtime` mount.

## Configuration

Build the image, then print a starting config:

```sh
docker compose build
docker run --rm drive-alpha \
    wcpan.drive.server init --plugin synology > drive-alpha.yaml
```

Paths in the config are container paths:

- `server.host: 0.0.0.0` (the default `127.0.0.1` is unreachable from the host)
- `server.database_url: /mnt/data/mirror.db`
- `local_paths` map Synology paths to `/mnt/watch/...`

To run next to the stable server against the same NAS:

- use a separate `DRIVE_DATA_PATH`; the alpha database schema is not
  compatible with the stable one
- set `webhook_callback_url` to the URL that reaches this project's
  `DRIVE_PORT`
- set a distinct `webhook_app_id` (for example `wcpan-drive-alpha`) so the two
  webhook registrations do not replace each other

## Usage

From this project directory:

```sh
docker compose config --quiet
docker compose build
docker compose up -d
docker compose logs --tail=100
```

To test newer code, rebuild without cache so the source is fetched again:

```sh
docker compose build --no-cache && docker compose up -d
```

Maintenance commands must run while the server is stopped:

```sh
docker compose stop
docker compose run --rm drive-alpha gc
docker compose run --rm drive-alpha backend synology backfill /photos
docker compose start
```
