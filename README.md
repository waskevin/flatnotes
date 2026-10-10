# Flatnotes for TerraMaster TOS 7

TOS 7 App Center package for [Flatnotes](https://github.com/dullage/flatnotes) — a database-less, self-hosted note-taking web app.

一个把 [Flatnotes](https://github.com/dullage/flatnotes) 打包上架到 TOS 7 应用中心的 Docker 应用（单容器、单数据卷、纯 Markdown 文件，无数据库）。

---

## What it does / 这是什么

Flatnotes keeps every note as an ordinary Markdown file in one flat folder on your TNAS — no
notebooks, no proprietary format, no database.

- Notes are plain Markdown files you can move anywhere at any time
- Full-text search (keyboard shortcut `/`), tags and wikilinks (`[[Another note]]`)
- Raw and WYSIWYG markdown editing modes, light and dark themes
- Mobile-responsive web interface, REST API
- Web UI on host port `10003` → container port `8080`
- No external database, no root, no privileged mode

## Package contents / 包内文件

```
flatnotes.tar.gz
├── config.ini            # TOS application metadata
├── flatnotes.lang        # 14 languages (zh-cn, zh-hk, en-us, ... pt-pt)
├── flatnotes.svg         # application icon (SVG, transparent background)
└── docker-compose.yml    # container definition
```

## docker-compose.yml

```yaml
version: "3.8"

services:
  flatnotes:
    image: dullage/flatnotes:v5.5
    restart: unless-stopped
    user: "1000:1000"
    ports:
      - "10003:8080"
    volumes:
      - ./data:/data
    environment:
      - TZ=Asia/Shanghai
      - FLATNOTES_AUTH_TYPE=password
      - FLATNOTES_USERNAME=admin
```

`x-app-meta` declares `web.port: 10003` / `web.protocol: http`, as required for Docker applications with a Web UI.

## Design decisions

**Non-root execution.** The upstream entrypoint drops privileges through `gosu` when the container
starts as root. This package starts the container directly as `1000:1000`, so that branch is
skipped and the application runs as the non-root user from the very first process.

**Credentials.** No password and no session secret is hardcoded. On first start a random
administrator password and a random session secret key are generated inside the data volume
(`data/.admin_password`, `data/.secret_key`, mode `600`) and are never written to logs. The user
name is `admin`. Both paths are documented in the `important` note of `flatnotes.lang`. The
persisted secret key keeps login sessions valid across container restarts.

**Container name.** `container_name` is intentionally omitted, so Compose derives a globally
unique name (`<project>_<service>_1`). The platform only appends a numeric suffix to a conflicting
*project* name — it does not rewrite `container_name` — so hard-coding a container name would make
installation fail on a device that already runs a container called `flatnotes`.

## Data location / 数据位置

The host side of `./data` is resolved by the platform to the application data root:

```
/Volume<N>/DockerAppData/flatnotes/data
```

Every note is a `.md` file in that directory; the search index lives in `data/.flatnotes`. Data
survives uninstall and reinstall as long as "delete data" is not selected.

## Build

```bash
./build.sh            # produces flatnotes.tar.gz + flatnotes.tar.gz.sha256
```

The archive must contain the four files at its root:

```bash
tar -czf flatnotes.tar.gz config.ini flatnotes.lang flatnotes.svg docker-compose.yml
sha256sum flatnotes.tar.gz > flatnotes.tar.gz.sha256
```

## Release

The package is published as a Release asset (the TOS Developer Platform downloads the package from
the Release, not from the repository root):

- Tag: `1.0.0`
- Asset: `flatnotes.tar.gz`
- Checksum: `flatnotes.tar.gz.sha256`

## Compliance notes / 合规说明

- Image source: Docker Hub only (`dullage/flatnotes`), version tag pinned, never `:latest`.
- Runs as a non-root user (`user: "1000:1000"`); no `privileged`, no `network_mode: host`.
- No hardcoded credentials, tokens or secrets in the image or the compose file.
- No secrets are written to container logs.
- The icon in this repository is original artwork created for this package; it is **not** the
  upstream project logo, to avoid trademark issues.
- This repository contains packaging metadata only. The application itself is developed by the
  Flatnotes project and distributed under its own license (MIT). All credit belongs to the upstream
  authors.
- Ports 22, 80, 443, 445, 3306, 5050, 5432, 6379, 8181 and 8443 are not used.

## Links

- Upstream project: https://github.com/dullage/flatnotes
- Docker Hub image: https://hub.docker.com/r/dullage/flatnotes
- TOS 7 development guide: https://github.com/terramaster-tos/tos-app-pkg-tools
