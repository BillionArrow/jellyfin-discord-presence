# Jellyfin Discord Rich Presence

A lightweight, CLI-only Discord Rich Presence client for Jellyfin. This [fork](https://github.com/kennethsible/jellyfin-rpc) bypasses external API limits by fetching 16:9 episode thumbnails directly from your Jellyfin instance, ensuring flawless metadata matching, even for alternate TVDB viewing orders (TMDB lacks alternate/dvd/etc order).

Jellyfin RPC updates your Discord status with what you're watching or listening to on your Jellyfin server. Make sure your Discord client is open while using Jellyfin RPC.

## Prerequisites & Installation

1. **Install `uv` (Package Manager)**  
   If you don't already have `uv` installed, run:
   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```

2. **Clone the Repository**
   ```bash
   git clone https://github.com/BillionArrow/jellyfin-rpc.git
   cd jellyfin-rpc
   ```

3. **Install Dependencies**
   ```bash
   uv sync
   ```

## Configuration

To generate a Jellyfin API key, go to the server dashboard and select **API Keys** under **Advanced**.

- Jellyfin Host (e.g., <http://192.168.1.100:8096>)
- Jellyfin API Key
- Jellyfin Username
- TMDB API Key (Optional)
- Public Server URL (Optional, e.g., <https://jellyfin.yourdomain.com>)

If you provide a **Public Server URL**, the plugin will serve images and 16:9 episode thumbnails directly from your Jellyfin instance—bypassing TMDB/TVDB rate limits completely and perfectly matching your library's artwork and episode orders. Click-through links will also prioritize TVDB if you use it for metadata.

To run the RPC, you must fill out an INI configuration file. You can find a template in [jellyfin_rpc.ini](https://github.com/BillionArrow/jellyfin-rpc/blob/main/jellyfin_rpc.ini). If you run into any issues, please change `log_level` in the INI to `DEBUG` and include the output in your GitHub Issue.

To fetch posters and album covers, your media must be properly tagged with the appropriate provider IDs.

> [!IMPORTANT]
> [**TMDB**](https://www.themoviedb.org/) is used to fetch posters for movies and TV shows as a fallback if you don't use the Public Server URL feature. However, if your media is scraped using **TVDB**, this script will prioritize linking to TVDB to prevent any mismatch issues (like alternate/dvd viewing orders). [**MusicBrainz**](https://musicbrainz.org/) and the [**Cover Art Archive**](https://coverartarchive.org/) are used to fetch album covers.

- `poster_languages` is a space- or comma-separated list of two-letter language codes ([ISO 639-1](https://en.wikipedia.org/wiki/ISO_639-1)) that indicates the preferred language(s) for TMDB posters.
- `season_over_series` controls whether season posters are preferred over series posters for shows.
- `release_over_group` controls whether release album covers are preferred over group album covers for music. The distinction between [release](https://musicbrainz.org/doc/Release) and [release group](https://musicbrainz.org/doc/Release_Group) is described in the MusicBrainz documentation. In short, a release is a specific version *or release* of an album that belongs to a *release group* (one per album).
- `find_best_match` searches for missing TMDB or MusicBrainz IDs before fetching posters or covers, respectively. Ideally, you should make sure that Jellyfin has populated these metadata tags.
- `show_when_paused` shows the rich presence with a paused timer instead of a progress bar. If disabled, the rich presence stops displaying when you pause your media.
- `show_server_name` shows your server name as the rich presence activity instead of saying Jellyfin.
- `show_jellyfin_icon` shows a small Jellyfin icon in the bottom right of the poster or album cover.

## Usage

To test the application in the foreground, simply run:

```bash
uv run jellyfin-rpc --ini-path jellyfin-rpc.ini
```

### Running as a Background Service (systemd)

To keep the Discord RPC running automatically in the background whenever you log in, you can set it up as a `systemd` user service on Linux.

1. Create a service file at `~/.config/systemd/user/jellyfin-rpc.service`:

```ini
[Unit]
Description=Jellyfin Discord RPC Background Service
After=network.target

[Service]
Type=simple
# Update WorkingDirectory if you cloned the repo somewhere else!
WorkingDirectory=%h/git/jellyfin-rpc
# %h automatically resolves to your home directory (e.g., /home/username)
ExecStart=%h/.local/bin/uv run jellyfin-rpc --ini-path ./jellyfin-rpc.ini

Restart=always
RestartSec=10

[Install]
WantedBy=default.target
```

2. Reload systemd, enable, and start the service:

```bash
systemctl --user daemon-reload
systemctl --user enable --now jellyfin-rpc.service
```

3. You can check the logs anytime with:

```bash
journalctl --user -fu jellyfin-rpc.service
```
