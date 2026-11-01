# BCKMon

🇬🇧 English · 🇪🇸 [Castellano](README.es.md)

**Backup monitor.** A read-only PWA that tells you whether your backups really ran: it reads the notification
emails (IMAP), checks files on a network disk (NFS) and warns you through push, email, Telegram, webhook or
Microsoft Teams when a backup fails, is missing or has not arrived on time.

BCKMon is the **backups-only edition of [PVEMon](https://github.com/Dagarin-es/PVEMon)**: both are built from the
same code. BCKMon does not connect to Proxmox or Docker and does not scan your network. If you also want to
monitor Proxmox servers, VMs, containers and disks, use PVEMon.

> **Beta.** The 2.0 series is published as betas (`dagarines/bckmon:2.0.0-beta.N`). On Docker Hub `latest` only
> moves when a stable version is released. The change history is in [`public/changelog.en.md`](public/changelog.en.md)
> (also behind the **ChangeLog** button in the menu).

## What it includes

| Area | Summary |
|---|---|
| **Backups by email** | Several IMAP accounts. Built-in format for **Synology Hyper Backup**; your own formats (Cobian, Duplicati, Veeam…) that you can edit and duplicate. Sources and tasks are created from the emails. |
| **File checks (NFS)** | Looks at a file, a pattern (`copia_*.tar`) or the newest file of a folder on a network disk: size and age. It does not read or copy it. |
| **Three views** | Settings › Backups: by email, by file, or grouped by source. |
| **Alerts** | Editable rules, per-task limits, snooze/reactivate, notification when they fire and when they recover. |
| **Notifications** | Push (Web Push), SMTP email, Telegram, generic JSON webhook and Teams (Workflows). |
| **Kiosk** | Panels for a wall screen. |
| **Users** | Administrator and monitor roles, TOTP 2FA, audit log. |
| **Languages** | English and Spanish, per user. |
| **System** | Encrypted backup, heartbeat (Healthchecks, Uptime Kuma…), export/import of backup settings with PVEMon (`.bckmonx`). |

## Installation

Image: [`dagarines/bckmon`](https://hub.docker.com/r/dagarines/bckmon) (amd64 and arm64). Create a folder with
the `docker-compose.yml` from [`repo-bckmon/docker-compose.yml`](repo-bckmon/docker-compose.yml) (it uses port
3001 so it can live next to PVEMon) and run:

```bash
echo "SECRET_KEY=$(openssl rand -base64 32)" > .env     # KEEP IT: it encrypts the stored secrets
mkdir -p data
docker compose up -d
docker compose exec bckmon node dist/cli.js create-admin admin   # shows the password ONCE
curl localhost:3001/api/health
```

- **Keep `SECRET_KEY`**: it encrypts stored passwords; without it they cannot be decrypted.
- Recover access: `docker compose exec bckmon node dist/cli.js reset-password admin`.
- Update: `docker compose pull && docker compose up -d` (migrations are automatic; keep `data/` and `.env`).
- The container runs as **root** (needed so the NFS check can come from a reserved port).
- Behind a reverse proxy (e.g. Nginx Proxy Manager): `http`, host IP, port `3001`, Websockets on; push and
  installing as an app require HTTPS. Close the port to the rest of the network.
- Variables: see the table in the [PVEMon README](README-PVEMON.md#environment-variables); `EDITION=bckmon` is already set in the image.

## Using PVEMon and BCKMon together

You can run both on the same machine on different ports with **separate data folders**. To move your backup
configuration from one to the other, use **Settings › System** in each: PVEMon exports and BCKMon imports, and the
other way round. The file is encrypted with a passphrase you choose.

## Source and Issues

This repository is a mirror of the same code as PVEMon, published so BCKMon has its own Issues and releases.
Code changes are made in the main project and published to both.
