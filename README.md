# Ferrite Unraid Templates

Unraid Community Applications template repository for **Ferrite** — a Rust DNS
sinkhole and filtering API server. It blocks ads and trackers at the DNS level,
supports encrypted upstream resolvers (DoH/DoT/DoQ), stores query statistics,
and ships a web/API management surface.

The template pulls the container image from the GitHub Container Registry:

```text
ghcr.io/syntlyx/ferrite-server:latest
```

## Install via Community Applications

Once the template is approved in Community Applications:

1. Open the **Apps** tab in Unraid and search for **Ferrite**.
2. Click **Install**. The template pre-fills ports, the appdata path, and the
   advanced variables described below.
3. Pick a network. Port 80 on the Unraid host is usually taken by the Unraid web
   UI, so either map the **Web UI** port to a free host port (e.g. `8088`) or
   give the container a dedicated IP on a custom Docker network / `br0`.
4. Click **Apply**. On first start the container downloads the Ferrite server
   binary and web UI from GitHub releases (verified by SHA-256) into appdata,
   then starts serving DNS on port 53 and the web UI on the mapped port.
5. Open the **WebUI** link from the Docker tab to reach the dashboard.

### Manual install (before CA approval)

Copy `ferrite/ferrite.xml` to a user-template file on the Unraid flash drive:

```text
/boot/config/plugins/dockerMan/templates-user/my-Ferrite.xml
```

Then open **Docker → Add Container** and select the **Ferrite** user template.

## Configuration reference

| Setting | Container target | Default | Notes |
| --- | --- | --- | --- |
| Appdata | `/var/lib/ferrite` | `/mnt/user/appdata/ferrite` | All persistent state (config, DB, caches, snapshots, web assets). |
| DNS TCP | `53/tcp` | `53` | DNS listener (TCP). |
| DNS UDP | `53/udp` | `53` | DNS listener (UDP). |
| Web UI | `80/tcp` | `80` | API + web UI. Remap to a free host port if 80 is taken. |
| Server Release Version | `FERRITE_SERVER_VERSION` | `latest` | `latest` auto-updates on restart; pin (e.g. `0.1.1`) to freeze. |
| Web Release Version | `FERRITE_WEB_VERSION` | _(blank)_ | Follows the server version when blank; pin separately if needed. |
| Panel IP | `FERRITE_PANEL_IP` | _(blank)_ | LAN IP for the built-in `fe.te` A record. Set to the Unraid host IP in bridge mode. |
| Panel URL | `FERRITE_PANEL_URL` | _(blank)_ | Startup-log shortcut URL. Set to `http://fe.te:<port>` when Web UI is not on host port 80. |
| GitHub Release Token | `FERRITE_RELEASE_TOKEN` | _(blank)_ | Optional. Only needed if you hit GitHub's anonymous API rate limit (60/hour per IP) or pull from a private mirror. Masked. |

### Health status

The container ships a Docker `HEALTHCHECK` that probes the public
`GET /api/auth` endpoint. Unraid surfaces this as the container health indicator
on the Docker tab — green once the API is serving, with a generous start period
so the first-start release download does not flap the status.

## Persistence

The template mounts one appdata directory:

```text
/mnt/user/appdata/ferrite -> /var/lib/ferrite
```

Ferrite stores all mutable state below that appdata path:

- Config: `/var/lib/ferrite/.config/ferrite/config.toml`
- SQLite database: `/var/lib/ferrite/ferrite.db`
- Blocklist cache and snapshots: `/var/lib/ferrite/.local/share/ferrite`
- Web UI assets: `/var/lib/ferrite/web`

Updating the Docker image should not remove settings, database history, caches,
or web assets.

The Community Applications icon is exported from the Ferrite web UI mark as
`icons/ferrite.png` so the Unraid listing matches the dashboard branding.

## Networking notes

Ferrite listens on container port 80 for the API/web UI. A custom Docker network
or fixed IP is recommended on Unraid because the Unraid web UI often already
uses host port 80.

When using bridge networking, set the advanced **Panel IP** variable to the
Unraid host/LAN IP if you want Ferrite's built-in `fe.te` DNS shortcut to
resolve outside the container. If the Web UI host port is not 80, open
`http://fe.te:<web-ui-host-port>` or set the advanced **Panel URL** variable to
the same URL so startup logs show the reachable address.

For client hostnames in bridge networking, configure Ferrite's reverse-DNS zone
to point at the router DNS. For a `192.168.1.0/24` LAN with router
`192.168.1.1`, patch settings with:

```bash
curl -sS -X PATCH http://<unraid-ip>:<web-ui-port>/api/settings \
  -H 'Content-Type: application/json' \
  -d '{"zones":[{"name":"1.168.192.in-addr.arpa","upstream":"192.168.1.1:53"}]}'
```

## Screenshots

Listing screenshots live under `screenshots/` and are referenced from the
Community Applications listing. Add PNGs captured from the running web UI:

| File | Suggested view |
| --- | --- |
| `screenshots/dashboard.png` | Dashboard with query/blocked totals and timeseries. |
| `screenshots/queries.png` | Live query log with client/status filters. |
| `screenshots/settings.png` | Settings → upstream resolvers and zone routing. |

<!-- Once captured, embed them here, e.g.:
![Ferrite dashboard](screenshots/dashboard.png)
-->
