# Media Stack

My Docker Compose setup for automating a Jellyfin media library. It is split into separate stacks so each can be started, updated and torn down on its own. Change it to your liking, not everyone has the same requirements.

Forked from [Wh1rr/ultimate-jellyfin-stack](https://github.com/Wh1rr/ultimate-jellyfin-stack).

## Stacks and services

| Folder | Service | Port | What it does |
|---|---|---|---|
| `qbit/` | **qBittorrent** | 8080 | BitTorrent client |
| `arrstack/` | **Radarr** | 7878 | Movie management and automation |
| | **Sonarr** | 8989 | TV show management and automation |
| | **Lidarr** | 8686 | Music management and automation |
| | **Prowlarr** | 9696 | Indexer manager for the *arr apps |
| | **Seerr** | 5055 | Lets users request media (successor to Jellyseerr) |
| | **Bazarr** | 6767 | Subtitle management for movies and TV shows |
| | **Homarr** | 80 | Dashboard with *arr, torrent and Jellyfin integrations |
| | **MeTube** | 8081 | Web UI for yt-dlp, downloads into `ytd/` |
| | **Maintainerr** | 6246 | Rule-based cleanup of watched/old media |
| `arcane/` | **Arcane** | 3552 | Docker GUI, manages the stacks in this repo as projects and keeps their images updated |

Unpackerr and Decluttarr are included in `arrstack/compose.yml` but commented out.

Jellyfin itself is not part of this repo.

## Dependencies

1. Linux
2. Docker / Docker Compose

## How to use

1. Create the shared network once. All stacks join it, so they can reach each other by container name (e.g. `http://qbittorrent:8080`):
   ```sh
   docker network create starr
   ```
2. In each stack folder copy `.env.example` to `.env` and fill it in. `.env` files are git-ignored.
3. Start the stacks:
   ```sh
   docker compose -f qbit/compose.yml up -d
   docker compose -f arrstack/compose.yml up -d
   docker compose -f arcane/compose.yaml up -d
   ```

Arcane mounts the repo checkout (`PROJECTS_DIR`), so every stack folder shows up there as a project you can start, stop and redeploy. The default login is `arcane` / `arcane-admin` and has to be changed on first sign-in.

To keep the images updated, enable **Image Update Watcher** and **Auto Update** under Environments → Automations. Arcane itself is updated from the Updates page with **Update All**.

## Environment variables

- `IP`: host IP the web UIs bind to
- `PUID` / `PGID`: user and group the containers run as
- `BASE_PATH`: where app configs are stored, e.g. `~/media/config`
- `MEDIA_SHARE`: root folder for downloads and media, e.g. `~/media/data`

See each stack's `.env.example` for the full list.

## Folder structure and hardlinks

To allow hardlinking to work (which you will definitely want!) every container must see downloads and media under the same root folder. Here `${MEDIA_SHARE}` is mounted as `/data` everywhere, so in the container a path looks like `/data/downloads/tv` or `/data/media/tv`.

An example folder structure:
![image](https://github.com/DonMcD/ultimate-plex-stack/assets/90471623/2003ac26-a929-4ff6-ad67-e35fc51fb51a)

Feel free to add folders like `music` as you need for your setup.

## Starr apps

Setting up the starr apps might be a bit confusing the first time, but to keep it simple:
1. Prowlarr manages the indexers and syncs them to Radarr, Sonarr and Lidarr
2. Those apps send their downloads to qBittorrent:
   1. Radarr - Movies
   2. Sonarr - TV Shows
   3. Lidarr - Music
3. Seerr combines the request systems of Radarr and Sonarr in a nice UI to find and auto download content.
4. Maintainerr removes old or watched media. Set its rules to unmonitor what they delete, otherwise Radarr/Sonarr will download it again.

Anytime you reference your media folder in a container you want the path to look like `/data/media/tv` instead of `/tv` like a lot of the default guides say. If you map the path as `/tv` hardlinking will not work.

## Recommendations

1. Get familiar with reverse proxies
2. Use a VPN for torrent traffic (e.g. Gluetun)
3. For remote access prefer Tailscale over exposing ports
4. Back up `${BASE_PATH}` regularly, Arcane updates the images automatically
