# Media Stack

My Docker Compose setup for a self-hosted media library, split into separate stacks. Change it to your liking.

Forked from [Wh1rr/ultimate-jellyfin-stack](https://github.com/Wh1rr/ultimate-jellyfin-stack).

## Stacks

- `qbit/`: download client
- `arrstack/`: media management and automation
- `arcane/`: Docker management

## How to use

Requires Linux and Docker Compose.

1. Create the shared network once:
   ```sh
   docker network create starr
   ```
2. In each stack folder copy `.env.example` to `.env` and fill it in.
3. Start the stacks:
   ```sh
   docker compose -f qbit/compose.yml up -d
   docker compose -f arrstack/compose.yml up -d
   docker compose -f arcane/compose.yaml up -d
   ```

Every container mounts `${MEDIA_SHARE}` as `/data` so hardlinks work. Use `/data/...` paths in each app.
