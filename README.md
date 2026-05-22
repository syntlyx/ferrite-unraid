# Ferrite Unraid Templates

Unraid Community Applications template repository for Ferrite.

Ferrite is a Rust DNS sinkhole and filtering API server. The template pulls the
container image from GitHub Container Registry:

```text
ghcr.io/syntlyx/ferrite-server:latest
```

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

The Community Applications icon is copied from the Ferrite web UI mark
(`ferrite-mark.svg`) so the Unraid listing matches the dashboard branding.

Ferrite listens on container port 80 for the API/web UI. A custom Docker network
or fixed IP is recommended on Unraid because the Unraid web UI often already
uses host port 80.

## Manual test

Before Community Applications approves the repository, copy
`ferrite/ferrite.xml` to:

```text
/boot/config/plugins/dockerMan/templates-user/ferrite.xml
```

Then open Unraid Docker -> Add Container and select the Ferrite user template.
