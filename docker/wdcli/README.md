# wdcli Docker Compose Project

Runs `wdcli` in a container using Docker Compose.

## Features

- Python 3.14 (slim-trixie) base image
- uv package manager installed
- libmediainfo0v5 system library for pymediainfo
- wdcli command available in container
- Volume mounts for config, data, storage, and the host timezone

## Setup

1. Copy the environment file and configure your paths:
   ```bash
   cp .env.example .env
   ```

2. Edit `.env` to set your absolute host paths:
   - `CONFIG_PATH`: Location of your config files (read-only)
   - `DATA_PATH`: Location of your data files (read-write)
   - `STORAGE_PATH`: Location of your storage files (read-write)

Create the configured host directories before building or running, and give the container user write access to `DATA_PATH` and `STORAGE_PATH`. All bind sources must already exist; Compose will fail instead of creating missing directories.

3. Build the image:
   ```bash
   docker-compose build
   ```

## Usage

Execute commands on-demand:
```bash
docker-compose run --rm shell wdcli --help
```

Or enter the shell:
```bash
docker-compose run --rm shell bash
```

The container will stop automatically after the command completes.

## Volume Mounts

- `/mnt/config` - Configuration files (read-only)
- `/mnt/data` - Data files (read-write)
- `/mnt/storage` - Storage files (read-write)
- `/etc/localtime` - Host timezone (read-only)

## Notes

- The wdcli script uses `uvx` to run wcpan.drive.cli with dependencies
- Config file is expected at `/mnt/config/cli.yaml`
- Container runs with `tail -f /dev/null` to stay alive
