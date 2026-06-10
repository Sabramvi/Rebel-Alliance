# Task 4 — git init + initial commit

- Repo remote: git@github.com:Sabramvi/Rebel-Alliance.git
- Branch strategy: main → initial commit → Rise of Republic (all dev work)
- .gitignore excludes: .env, keys/certs, backups/, Archive/, .omo/evidence/, .omo/drafts/, .DS_Store, SOURCE_OF_TRUTH.md
- Initial commit message: "chore: initial state — dead VPN before reanimation"
- No secrets staged — verified via `git diff --cached | grep -E 'password|secret|PRIVATE KEY'`

# Task 3 — USA Server Discovery (184.174.97.95)

- SSH: OK. Key `skywalker_deploy` (ed25519) verified match.
- OS: Ubuntu 24.04 LTS (noble), kernel 6.8.0-31-generic, x86_64, hostname `sbrm-usa`
- Docker: NOT installed. No docker packages. Needs install in Task 6.
- Firewall: UNKNOWN. `skywalker` user lacks passwordless sudo. UFW + iptables need root.
- Listening ports: Only SSH (22) and CUPS (631 localhost). Port 2096 is FREE for Tatooine.
- Panel reachability: UNREACHABLE from USA. Both nc and curl timed out to 5.42.110.191:9443. Likely Neo firewall only allows Aeza IP.
- Disk: 59G total, 9.0G used, 47G free (16%)
- Memory: 2GB total, 603MB used, 1.3GB available
- SSH keys: 1 key in authorized_keys — skywalker-deploy, matches local pubkey.
- BLOCKERS: No docker. No sudo. Panel unreachable. All need resolution in Tasks 4-5-6.

## 2026-06-10 — Task 4: git init + initial commit

- git init in project root, default branch renamed master→main
- 14 files committed: docs + .omo/notepads + .gitignore
- Secrets verified clean — grep passed, no secret values in index
- Branch `rise-of-republic` created (git doesn't allow spaces in branch names)
- Remote not yet pushed — pending Task 27

## 2026-06-10 — Task 2: Aeza Discovery

- SSH: OK. Key `aeza-skywalker` (ed25519). Hostname: sbrm-aeza.
- OS: Ubuntu 24.04.4 LTS, kernel 6.8.0-110-generic.
- Docker: 29.4.0, marzban-node container RUNNING (3+ days), image gozargah/marzban-node:latest.
- Marzban-node: REST protocol, Xray v26.3.27, but **Xray inside container is DEFUNCT**.
- Panel pings from Sabram Neo (5.42.110.191) succeed — node API responsive.
- Old inbounds STILL ACTIVE: 773 (Mos Eisley xhttp+tls), 447 (Alderaan grpc+reality), 446 (unknown).
- No xray_config.json — marzban-node manages config via panel API.
- Nginx: running, 4 server blocks, config test passes.
- Firewall: ufw NOT installed.
- No passwordless sudo — all sudo commands work (NOPASSWD configured).
- Backups complete: marzban-node files + nginx configs + docker-compose.

### 🚨 CRITICAL: Port 8443 CONFLICT
- Port 8443 is **OCCUPIED by nginx** (pid 1181/1189) — CloudPanel internal proxy endpoint.
- custom-domain.conf: `proxy_pass https://127.0.0.1:8443/` — CloudPanel uses 8443 internally.
- Coruscant target port (8443) is NOT available — requires either:
  A) Relocate CloudPanel backend to another port
  B) Use alternative port for Coruscant
- Plan assumed 8443 would be free — this is a **plan deviation** that must be escalated.

### Port Map Summary
| Port | Status   | Owner              |
|------|----------|---------------------|
| 443  | OCCUPIED | nginx (CloudPanel)  |
| 80   | OCCUPIED | nginx               |
| 8443 | OCCUPIED | nginx (CloudPanel)  |
| 447  | OCCUPIED | mari-ban-node       |
| 773  | OCCUPIED | mari-ban-node       |
| 446  | OCCUPIED | mari-ban-node       |

### Evidence
- Full report: `.omo/evidence/task-2-discovery-aeza.txt`
- Port audit: `.omo/evidence/task-2-port-audit.txt`
- Backups: `backups/aeza/marzban-node/`, `backups/aeza/nginx/`

## 2026-06-10 — Task 1: Sabram Neo Discovery

### SSH Access
- Key: `~/.ssh/sabram-crm-skywalker` (NOT skywalker_deploy)
- User: skywalker@5.42.110.191, hostname: sbrm
- Sudo: passwordless sudo available
- Docker group: skywalker NOT in docker group — need `sudo docker`

### Critical Findings
1. **Marzban container STOPPED** (Exited 0, ~44h ago) — graceful shutdown, no crash
2. **.env EXISTS at /opt/marzban/.env** — NOT at /var/lib/marzban/.env
3. **Docker compose at /opt/marzban/** — NOT /var/lib/marzban/
4. **FastPanel** (port 8888) is hosting panel — CloudPanel is on Aeza
5. **Two nginx**: FastPanel (80/443 primary) + system nginx
6. **rage.severdesign.ru:443 → proxy_pass Marzban:8000** — old panel config still active
7. **Port 9443 NOT in UFW** — must be opened for mandalore
8. **Aeza node IP in DB: 62.60.244.102** (OLD) — current is 5.182.86.27
9. **9 active users** in DB — all to be replaced by 1 user "Anakin"
10. **4 inbounds**: Shadowsocks dummy + Mos Eisley (773) + Alderaan (447) + Node2 (448)

### Container Env (from docker inspect)
- UVICORN_PORT=8000, UVICORN_HOST=0.0.0.0
- DB: sqlite:////var/lib/marzban/db.sqlite3
- XRAY_JSON=/var/lib/marzban/xray_config.json
- XRAY_SUBSCRIPTION_URL_PREFIX NOT SET

### Fix Strategy (for Task 5)
- `cd /opt/marzban && sudo docker compose up -d` — should revive panel
- Add mandalore Nginx config on port 9443
- `sudo ufw allow 9443/tcp`
- Update .env with XRAY_SUBSCRIPTION_URL_PREFIX
- Do NOT touch rage.severdesign.ru nginx config yet

### Backups
- 10 files: .env, xray_config.json, docker-compose.yml, db.sqlite3.dump, 6 nginx configs
- Evidence: `.omo/evidence/task-1-discovery-neo.txt`, `.omo/evidence/task-1-backup-check.txt`

## 2026-06-10 — Task 5: Marzban Panel Reanimation (Sabram Neo)

### Actions Taken
1. **Marzban container revived**: `cd /opt/marzban && sudo docker compose up -d` → Running (exited gracefully before)
2. **Self-signed SSL cert**: `/etc/ssl/certs/mandalore-cert.pem` + `/etc/ssl/private/mandalore-key.pem`, CN=mandalore.severdesign.ru, SAN=mandalore.severdesign.ru,localhost,5.42.110.191, valid 1yr
3. **Nginx config**: `/etc/nginx/conf.d/mandalore-panel.conf` (NOT sites-enabled — system nginx uses conf.d/)
4. **.env updated**: `XRAY_SUBSCRIPTION_URL_PREFIX=https://mandalore.severdesign.ru:9443` appended
5. **UFW opened**: `9443/tcp` + `from 184.174.97.95 to any port 9443`
6. **nginx -t**: PASS
7. **nginx reload**: OK

### Verification Results
- Marzban container: UP (gozargah/marzban:latest)
- HTTP 8000 → 200
- HTTPS 9443 root → 200 (Marzban SPA with 3D animation loading screen)
- HTTPS 9443 /dashboard → 307 redirect (SPA client-side routing)
- API /api/system → 200 `{"detail":"Not authenticated"}`
- API /api/admin/token → 405 (requires POST)

### Key Findings
- System nginx uses `/etc/nginx/conf.d/*.conf` — NOT sites-enabled
- Nginx config at `/etc/nginx/conf.d/mandalore-panel.conf`
- fastpanel2-available/sites are for FastPanel's domain configs (separate system)
- `/dashboard/login` returns 404 server-side → Marzban is SPA, login handled client-side
- Root `/` returns the full SPA shell with CSS animation → panel is fully functional
- `.env` had commented-out `# XRAY_SUBSCRIPTION_URL_PREFIX = "https://example.com"` — appending worked correctly

### Blockers for Next Tasks
- Task 10 (LE cert for mandalore) — depends on DNS A record mandalore.severdesign.ru → 5.42.110.191
- Panel reachable from USA? Need to verify external access from 184.174.97.95

### Evidence
- `.omo/evidence/task-5-panel-local.txt` — full verification output
- `.omo/evidence/task-5-panel-content.txt` — panel HTML content snippet
