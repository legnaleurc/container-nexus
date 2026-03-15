# Drive

## Setup

Copy `.env.example` to `.env` and configure the host paths and listening address.
Place the Drive configuration file at `DRIVE_CONFIG_FILE` and create the
`DRIVE_DATA_PATH` and `DRIVE_WATCH_PATH` directories before starting. Give the
container user write access to `DRIVE_DATA_PATH` and read access to the
configuration file and watch directory. All bind sources must already exist;
Compose will fail instead of creating missing directories.

The container uses the host timezone through a read-only `/etc/localtime` mount.

From this project directory:

```sh
docker compose config --quiet
docker compose build
docker compose up -d
docker compose logs --tail=100
```
