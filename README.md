# KaliForge

Kali Rolling cloud workspace for authorized labs and CTFs. Combines the Kali Railway reference with RailDK's SSH, 9router, Hermes, and split GitHub backups. Existing reference repositories remain untouched.

## Persistence

Railway volume mounted at `/root` is primary persistence across restarts/redeploys. Two private GitHub repositories provide a separate off-volume backup:

- `APPS_REPO=owner/repo`: `/root/.9router` + `/root/.hermes`. SQLite databases use `.backup`; transient files, logs, `.env`, auth/credential-named files are excluded. Review Hermes/provider configuration before enabling this; files not matched by these exclusions can still contain secrets.
- `KALI_REPO=owner/repo`: allowlisted `/root/tools`, `/root/wordlists`, `/root/labs`, and `.zshrc`, `.bashrc`, `.vimrc`, `.tmux.conf`. `.env`, `*.key`, and `*.pem` excluded. Does not back up the OS or apt-installed packages; Dockerfile rebuilds those.

Both repos must already exist and be private. Set `GITHUB_TOKEN` with contents read/write access. Restore runs before services; backups run every `SYNC_INTERVAL` seconds (default 300). Manual: `app-sync backup`, `kali-sync backup`; status: `app-sync status`, `kali-sync status`.

## Services

- `22/tcp`: SSH (Railway TCP proxy)
- `PORT/http`: ttyd browser terminal (Railway-provided `PORT`)
- `20128/http`: 9Router; do not publish publicly until management routes are independently authenticated. Its API key mechanism does not establish that the admin UI is protected.
- `8686/http`: Hermes dashboard (basic auth)
- `8642/http`: optional Hermes API (currently disabled by default)

Railway needs explicit public TCP proxy/domain routing for non-HTTP ports. Keep 9Router private; public dashboard/admin exposure is not verified safe.

`ROOT_PASSWORD` is required; startup fails closed if missing. It protects ttyd, SSH password login, and Hermes dashboard (dashboard password stored as scrypt hash). Set `AUTHORIZED_KEYS` too, then prefer disabling SSH password login once key access is verified. Do not expose unauthenticated services. Enable `hermes.serve_enabled` in `/etc/kaliforge/config.json` only if the mobile/API client needs it. SSH password authentication remains enabled even when `AUTHORIZED_KEYS` is set; use a strong `ROOT_PASSWORD`. Consider Railway resource limits: Kali + Metasploit + Node + Hermes may exceed small memory allocations. Attach a Railway volume at `/root` in the service settings; the config files do not create a volume.

## Variables

Required/optional: `ROOT_PASSWORD`, `AUTHORIZED_KEYS`, `GITHUB_TOKEN`, `APPS_REPO`, `KALI_REPO`, `SYNC_INTERVAL`, `USERNAME`, `PORT`. Attach the `/root` volume manually in Railway service settings. Hermes serve options are controlled in `/etc/kaliforge/config.json`, not by Railway variables.

## Build

```sh
docker build -t kaliforge .
```

## Security / scope

Use Kali tools only against systems you own or have explicit authorization to test. GitHub backup is an allowlisted workspace backup, not a full Kali image or package persistence. Inspect secrets before committing app state; keep credentials in Railway Variables, not Git or backup repositories.

ponytail: apt-installed packages persist across container restarts, not image rebuilds; add required package names to the Dockerfile package list when you need them after redeploy.
