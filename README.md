# Vaultwarden Homelab Password Manager

A self-hosted Bitwarden-compatible password manager server, deployed as a Docker stack through Portainer, running on WSL2 (Ubuntu 22.04).

## Why This Project

Running your own password manager backend touches real infrastructure concerns: persistent databases, backups, and why encryption at rest actually matters — not just abstract security terms, but a system where a bad backup strategy means losing real credentials. This project demonstrates deploying and understanding a data-persistent service, not just a stateless web app.

## Stack

- **Docker Desktop** (WSL2 backend)
- **WSL2** — Ubuntu 22.04
- **Portainer CE** — used to deploy this as a Stack (builds on the docker-portainer-homelab and uptime-kuma-homelab projects)
- **Vaultwarden** — lightweight, Bitwarden-compatible server implementation

## What I Built

1. Deployed Vaultwarden as a Portainer Stack using the Docker Compose definition below
2. Confirmed the web vault loads and a new account can be created
3. Verified data persists in a named Docker volume, independent of the container lifecycle
4. Documented the setup as a local-only demo (not exposed beyond localhost) — full external access is planned as a follow-on once Nginx Proxy Manager (Project 4) provides HTTPS

## How to Run It

**Option A — via Portainer (what I used):**

1. Open Portainer → Stacks → Add stack
2. Name it vaultwarden, paste in docker-compose.yml from this repo, and deploy
3. Open http://localhost:8081

**Option B — one command, no Portainer required:**

```
docker compose up -d
```

Either way, the web vault login/registration screen loads immediately at http://localhost:8081.

## Try It Yourself

There's no pre-configured demo login — that's intentional, not an oversight. Vaultwarden ships with a completely empty database on first run, because it's a zero-knowledge encryption system: even the server operator shouldn't be able to pre-seed or inspect vault contents. Shipping a database with a baked-in demo account (and its password hash) in this repo would work, but it's exactly the kind of thing security tooling flags, and it undercuts the point of the project.

To try it yourself: click Create account, register with any email/password, and you'll land in an empty vault dashboard. In a real deployment, you'd set SIGNUPS_ALLOWED=false in the environment once your own account exists, so the server stops accepting new registrations from anyone else who reaches it.

## Notes on Production Use

This deployment is intentionally local-only, and that's a deliberate security boundary, not a limitation I hit and stopped at. Bitwarden's official clients (and Vaultwarden itself) require a proper HTTPS context to complete login and encryption operations — the one exception being requests to the literal host localhost, which browsers treat as a "secure context" for local development. Running this over plain HTTP on any other address, or exposing it to a LAN/the internet without a certificate, would mean master passwords and vault data crossing the network unencrypted — which defeats the entire point of a password manager.

So rather than force a workaround (self-signed certs on a throwaway demo, disabling security checks, etc.), this repo stops at "server deployed and reachable" and picks the real fix back up in Project 4 (Nginx Proxy Manager), which provisions a free, auto-renewing HTTPS certificate. Once that's in place, this same Vaultwarden container gets a real domain and certificate in front of it, and full login/sync works exactly as it would in production — across every device, not just this one machine.

## Backups

Vaultwarden stores its data in a SQLite database inside the `vaultwarden_data` named volume. That database is the entire vault — encrypted entries, user records, and the metadata that ties them together. **A password manager with no backup strategy is a password manager that loses everything in a single disk failure.**

This repo runs local-only and does not implement automated backups — it is a deployment demo, not the storage layer you would trust with credentials you cannot afford to lose. Before using this for real, the minimum viable backup setup is:

- A nightly SQLite `.backup` snapshot (safe while the service is running, unlike a raw file copy of `db.sqlite3`)
- Copied off-host — to a NAS, object storage, or another machine
- With at least one generation retained beyond the most recent, so a corrupted backup does not overwrite a good one
- Tested by restoring into a scratch container at least once

The named-volume design in `docker-compose.yml` makes this straightforward: a `docker run --rm` invocation that mounts both `vaultwarden_data` and a backup directory can produce a portable tarball without stopping the container.

## Screenshots

| Step | Screenshot |
|------|------------|
| Portainer stack deployed | screenshots/01-stack-deployed.png |
| Vaultwarden web vault (login/registration screen) | screenshots/02-web-vault.png |

## Related Projects

This repo is one piece of a self-hosted homelab portfolio:

- **[docker-portainer-homelab](https://github.com/shepdogg6t7-glitch/docker-portainer-homelab)** — container management UI (the layer this stack deploys through)
- **[uptime-kuma-homelab](https://github.com/shepdogg6t7-glitch/uptime-kuma-homelab)** — service monitoring and alerting
- **[nginx-proxy-manager-homelab](https://github.com/shepdogg6t7-glitch/nginx-proxy-manager-homelab)** — reverse proxy and TLS termination (provides HTTPS for this service in a full deployment)

## Credit

Project idea from Joe (@joecoxtech) — "Build Your Way Into IT: Five Free Home Lab Projects."
