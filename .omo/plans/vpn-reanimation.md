# Skywalker-VPN: Reanimation

## TL;DR

> **Quick Summary**: Reanimate dead Marzban VPN — rename all domains, create 2 new inbounds (Coruscant GRPC REALITY + Tatooine XHTTP TLS), fix broken panel, add USA as 2nd data plane, apply strict routing for AI services, 1 subscription, 1 user, Tolaria wiki handoff.

> **Deliverables**:
> - Working Marzban panel on mandalore.severdesign.ru:9443
> - Coruscant inbound: VLESS GRPC REALITY on coruscant.severdesign.ru:8443 (Aeza)
> - Tatooine inbound: VLESS XHTTP TLS on tatooine.severdesign.ru:2096, SNI dagoba.severdesign.ru (USA)
> - Mask sites: jedha.severdesign.ru → hello, bespin.severdesign.ru → hello
> - 1 subscription "Rebel Alliance", 1 user "Anakin"
> - Strict routing: Adobe/OpenAI/Anthropic/Gemini/Antigravity → proxy, RU → direct
> - Git repo with branch "Rise of Republic", Tolaria knowledge base

> **Estimated Effort**: Large
> **Parallel Execution**: YES — 6 waves + Final
> **Critical Path**: Discovery → Fix Panel → Certs → Configure Inbounds → QA → Docs

---

## Context

### Original Request
User (Anakin): VPN is dead after failed agent changes. TODO.md is canonical — rename all domains, change all ports, add USA as 2nd data plane, fix everything, strict AI service routing.

### Interview Summary
**Key Discussions**:
- Ports: Mandalore 9443, Coruscant 8443, Tatooine 2096, masks 80 — APPROVED
- TLS certs: Let's Encrypt DNS challenge — CONFIRMED
- Panel strategy: Fix existing, don't reinstall — CONFIRMED
- USA server: Ubuntu 24.04, SSH key skywalker_deploy.pub, Docker maybe not installed
- All new DNS domains already created
- Agent to discover Marzban/Nginx/Xray state via SSH

**Research Findings**:
- Atlanta-xhttp-profile-june.json: comprehensive routing template (RU → direct, Apple push → direct, bittorrent → direct, ~450 RU domains)
- SSH keys for all 3 servers in Tolaria vault (deploy-keys.md)
- XRAY_CONFIG.current.json: old Mos Eisley (xhttp+tls:773) + Alderaan (grpc+reality:447) — both to be removed
- No test infrastructure exists (infra project)

### Metis Review
**Identified Gaps** (addressed):
- 4 subscriptions → 1: User explicitly wants "1 subscription Rebel Alliance, 1 user Anakin" — old users are the same person
- Port 9443: Commonly scanned, but user approved — noted as risk, proceed with user's choice
- TODO.md divergences: "Mandalore inbound" on USA is treated as panel on Sabram Neo (per target section); bespin mapped to USA not Aeza (user confirmed)
- Discovery-first approach: Mandatory — agent MUST discover server state before making changes
- Backup first: Mandatory — previous agent didn't backup and killed everything
- Firewall: USA→Sabram Neo connectivity must be verified

---

## Work Objectives

### Core Objective
Reanimate dead Marzban VPN with 2 new inbounds (Coruscant GRPC REALITY + Tatooine XHTTP TLS), renamed domains, clean ports, strict AI-service routing, and Tolaria wiki handoff.

### Concrete Deliverables
- Working panel: `https://mandalore.severdesign.ru:9443/dashboard/`
- Coruscant inbound: VLESS GRPC REALITY, coruscant.severdesign.ru:8443, serviceName `cor-grpc`
- Tatooine inbound: VLESS XHTTP TLS, tatooine.severdesign.ru:2096, SNI dagoba.severdesign.ru, path `/api`
- Jedha mask: HTTP redirect jedha.severdesign.ru:80 → hello.severdesign.ru
- Bespin mask: HTTP redirect bespin.severdesign.ru:80 → hello.severdesign.ru
- Subscription: `https://mandalore.severdesign.ru:9443/sub/...` with 2 inbounds
- Git repo: branch `Rise of Republic`, push to remote
- Tolaria: `Skywalker-VPN-wiki/` knowledge base

### Definition of Done
- [ ] `curl -sk https://mandalore.severdesign.ru:9443/dashboard/login` → 200 OK
- [ ] `openssl s_client -connect coruscant.severdesign.ru:8443 -servername coruscant.severdesign.ru </dev/null 2>/dev/null | openssl x509 -noout -subject` → CN = coruscant.severdesign.ru
- [ ] `openssl s_client -connect tatooine.severdesign.ru:2096 -servername dagoba.severdesign.ru </dev/null 2>/dev/null | openssl x509 -noout -subject` → CN = dagoba.severdesign.ru or tatooine.severdesign.ru
- [ ] `curl -sI http://jedha.severdesign.ru` → 301/302 Location: hello.severdesign.ru
- [ ] `curl -sI http://bespin.severdesign.ru` → 301/302 Location: hello.severdesign.ru
- [ ] Subscription URL returns JSON with 2 outbound configs
- [ ] Marzban panel shows 2 connected nodes (Aeza + USA)
- [ ] All 5 project docs updated, old files archived, .gitignore excludes secrets/Archive
- [ ] Tolaria wiki created with indexed notes
- [ ] Git pushed to remote on branch `Rise of Republic`

### Must Have
- Panel accessible on mandalore.severdesign.ru:9443
- Coruscant: VLESS GRPC REALITY on Aeza (5.182.86.27), port 8443
- Tatooine: VLESS XHTTP TLS on USA (184.174.97.95), port 2096, SNI dagoba
- Jedha mask redirect on Aeza (port 80)
- Bespin mask redirect on USA (port 80)
- 1 subscription "Rebel Alliance"
- 1 user "Anakin"
- Routing: Adobe, OpenAI, Anthropic, Gemini, Antigravity → proxy; RU domains → direct
- Git repo: initial commit + branch "Rise of Republic"
- Backup of all current configs BEFORE any changes
- Tolaria knowledge base at end

### Must NOT Have (Guardrails)
- Do NOT touch CloudPanel (on Aeza)
- Do NOT touch sabram.ru (on Aeza)
- Do NOT touch port 443 on Aeza (CloudPanel)
- Do NOT reinstall Marzban panel — fix only
- Do NOT use old ports 773, 447
- Do NOT keep old inbounds (Mos Eisley, Alderaan)
- Do NOT delete old users without confirmation
- Do NOT commit secrets/passwords/keys to git
- Do NOT commit Archive/ to git
- Do NOT proceed without backup first
- Do NOT skip discovery — must check server state before changes

---

## Verification Strategy

> **ZERO HUMAN INTERVENTION** — ALL verification is agent-executed. No exceptions.

### Test Decision
- **Infrastructure exists**: NO (infra project, no code test framework)
- **Automated tests**: None
- **Agent-Executed QA**: YES — every task verified via curl, openssl, SSH, docker exec, browser snapshot

### QA Policy
Every task MUST include agent-executed QA scenarios.
Evidence saved to `.omo/evidence/task-{N}-{scenario-slug}.{ext}`.

- **Panel/API**: curl — Send requests, assert status + response
- **TLS/SSL**: openssl s_client — Verify certs, handshake, SNI
- **Server state**: SSH + docker exec — Check container status, logs, configs
- **Mask redirects**: curl -sI — Follow redirects, assert Location header

---

## Execution Strategy

### Parallel Execution Waves

```
Wave 1 (Start Immediately — discovery + backup + git):
├── Task 1: SSH discovery + backup on Sabram Neo [deep]
├── Task 2: SSH discovery + backup on Aeza [deep]
├── Task 3: SSH discovery on USA [deep]
└── Task 4: Git init + initial commit [quick]

Wave 2 (After Wave 1 — server setup, MAX PARALLEL):
├── Task 5: Fix Marzban panel on Sabram Neo (depends: 1) [deep]
├── Task 6: Install Docker + Marzban-node on USA (depends: 3) [deep]
├── Task 7: Configure Aeza node for Coruscant (depends: 2) [deep]
├── Task 8: Nginx mask jedha → hello on Aeza (depends: 2) [quick]
└── Task 9: Nginx mask bespin → hello on USA (depends: 3, 6) [quick]

Wave 3 (After Wave 2 — certificates, ALL PARALLEL):
├── Task 10: LE cert mandalore on Sabram Neo (depends: 5) [deep]
├── Task 11: LE cert coruscant on Aeza (depends: 7, 8) [deep]
├── Task 12: LE certs tatooine + dagoba on USA (depends: 6, 9) [deep]
├── Task 13: LE cert jedha on Aeza (depends: 8) [quick]
└── Task 14: LE cert bespin on USA (depends: 6, 9) [quick]

Wave 4 (After Wave 3 — Marzban inbounds + routing + user):
├── Task 15: Configure Coruscant inbound on panel (depends: 10, 11) [deep]
├── Task 16: Configure Tatooine inbound on panel (depends: 10, 12) [deep]
├── Task 17: Remove old inbounds + old users (depends: 15, 16) [quick]
├── Task 18: Configure routing rules (depends: 15, 16) [deep]
└── Task 19: Create user Anakin + subscription Rebel Alliance (depends: 15, 16) [quick]

Wave 5 (After Wave 4 — QA, ALL PARALLEL):
├── Task 20: QA panel + subscription endpoint [unspecified-high]
├── Task 21: QA Coruscant inbound [unspecified-high]
├── Task 22: QA Tatooine inbound [unspecified-high]
├── Task 23: QA mask redirects [unspecified-high]
└── Task 24: QA AI-service routing via proxy [unspecified-high]

Wave 6 (After Wave 5 — docs + git + Tolaria):
├── Task 25: Update project docs [writing]
├── Task 26: Archive old files + .gitignore [quick]
├── Task 27: Git commit + push to Rise of Republic [quick]
└── Task 28: Tolaria knowledge base [writing]

Wave FINAL (After ALL — 4 parallel reviews, then user okay):
├── Task F1: Plan compliance audit (oracle)
├── Task F2: Code quality review (unspecified-high)
├── Task F3: Real manual QA (unspecified-high)
└── Task F4: Scope fidelity check (deep)
-> Present results -> Get explicit user okay

Critical Path: Task 1 → Task 5 → Task 10 → Task 15 → Task 20 → Task 25 → F1-F4
Parallel Speedup: ~65% faster than sequential
Max Concurrent: 5 (Waves 2, 3, 5)
```

### Dependency Matrix

| Task | Blocks | Blocked By | Wave |
|------|--------|------------|------|
| 1 | 5 | — | 1 |
| 2 | 7, 8 | — | 1 |
| 3 | 6, 9 | — | 1 |
| 4 | — | — | 1 |
| 5 | 10 | 1 | 2 |
| 6 | 9, 12, 14 | 3 | 2 |
| 7 | 11 | 2 | 2 |
| 8 | 11, 13 | 2 | 2 |
| 9 | 12, 14 | 3, 6 | 2 |
| 10 | 15, 16 | 5 | 3 |
| 11 | 15 | 7, 8 | 3 |
| 12 | 16 | 6, 9 | 3 |
| 13 | — | 8 | 3 |
| 14 | — | 6, 9 | 3 |
| 15 | 17, 18, 19 | 10, 11 | 4 |
| 16 | 17, 18, 19 | 10, 12 | 4 |
| 17 | — | 15, 16 | 4 |
| 18 | — | 15, 16 | 4 |
| 19 | — | 15, 16 | 4 |
| 20 | — | 19 | 5 |
| 21 | — | 15 | 5 |
| 22 | — | 16 | 5 |
| 23 | — | 13, 14 | 5 |
| 24 | — | 18 | 5 |
| 25 | — | 20-24 | 6 |
| 26 | — | 25 | 6 |
| 27 | — | 26 | 6 |
| 28 | — | 25 | 6 |

### Agent Dispatch Summary

- **Wave 1**: 4 — T1-T3 → `deep`, T4 → `quick`
- **Wave 2**: 5 — T5-T7 → `deep`, T8-T9 → `quick`
- **Wave 3**: 5 — T10-T12 → `deep`, T13-T14 → `quick`
- **Wave 4**: 5 — T15-T16 → `deep`, T17 → `quick`, T18 → `deep`, T19 → `quick`
- **Wave 5**: 5 — T20-T24 → `unspecified-high`
- **Wave 6**: 4 — T25 → `writing`, T26 → `quick`, T27 → `quick`, T28 → `writing`
- **FINAL**: 4 — F1 → `oracle`, F2 → `unspecified-high`, F3 → `unspecified-high`, F4 → `deep`

---

## TODOs

> Implementation + Test = ONE Task. Never separate.
> EVERY task MUST have: Recommended Agent Profile + Parallelization info + QA Scenarios.
> **A task WITHOUT QA Scenarios is INCOMPLETE. No exceptions.**
> **FORMAT**: Task labels MUST use bare numbers: `1.`, `2.`, `3.` — NOT `T1.`, `Task 1.`, `Phase 1:`.

- [x] 1. SSH discovery + backup on Sabram Neo (5.42.110.191)

  **What to do**:
  - SSH to skywalker@5.42.110.191
  - Check Docker: `docker ps -a`, `docker images | grep marzban`
  - Check Marzban container logs: `docker logs marzban-marzban-1 --tail 100`
  - Check Marzban directory: `ls -la /var/lib/marzban/`, check for `.env`, `xray_config.json`
  - Check Nginx: `nginx -t`, `ls /etc/nginx/sites-enabled/`, `nginx -T 2>/dev/null | grep -E 'server_name|listen|proxy_pass'`
  - Check DB: `ls -la /var/lib/marzban/db.sqlite3` or check if Postgres container exists
  - Check current inbounds via Marzban API if accessible: `curl -sk https://localhost:9443/api/inbounds` or check panel DB
  - Backup ALL configs: `/var/lib/marzban/` → copy relevant files to local backup
  - Backup Nginx configs: `/etc/nginx/sites-enabled/` → copy to local
  - Check firewall: `ufw status`, `iptables -L -n | head -30`

  **Must NOT do**:
  - Do NOT restart any services
  - Do NOT modify any configs
  - Do NOT delete anything

  **Recommended Agent Profile**:
  - **Category**: `deep`
    - Reason: Complex multi-service discovery requiring careful investigation and understanding
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed — pure SSH exploration

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 2, 3, 4)
  - **Blocks**: Task 5
  - **Blocked By**: None (can start immediately)

  **References**:
  - `TOPOLOGY.md` — Server roles and IPs
  - `CURRENT_STATE.md` — Known broken state description
  - `OPERATIONS_AND_ROLLBACK.md` — Rollback commands reference
  - `deploy-keys.md` (Tolaria) — SSH connection strings
  - `XRAY_CONFIG.current.json` — Old config snapshot for comparison

  **Acceptance Criteria**:
  - [ ] Report produced: Docker state (running/stopped/missing), Marzban version, Nginx state, DB state, current inbounds list, firewall rules
  - [ ] Backup files saved to project: `backups/sabram-neo/` directory

  **QA Scenarios**:
  ```
  Scenario: Discovery report completeness
    Tool: Bash (SSH)
    Preconditions: SSH access to skywalker@5.42.110.191 works
    Steps:
      1. ssh skywalker@5.42.110.191 'docker ps -a 2>/dev/null || echo "NO_DOCKER"'
      2. ssh skywalker@5.42.110.191 'docker logs marzban-marzban-1 --tail 50 2>/dev/null || echo "NO_LOGS"'
      3. ssh skywalker@5.42.110.191 'ls /var/lib/marzban/ 2>/dev/null || echo "NO_MARZBAN_DIR"'
      4. ssh skywalker@5.42.110.191 'nginx -t 2>&1 || echo "NO_NGINX"'
    Expected Result: All 4 checks return meaningful output (not all "NO_*")
    Failure Indicators: SSH connection fails, all checks return "NO_*" — server may be unreachable or completely bare
    Evidence: .omo/evidence/task-1-discovery-neo.txt

  Scenario: Backup files exist locally
    Tool: Bash (local)
    Steps:
      1. ls backups/sabram-neo/
      2. Count files: ls backups/sabram-neo/ | wc -l
    Expected Result: At least 3 backup files from /var/lib/marzban/ and /etc/nginx/
    Evidence: .omo/evidence/task-1-backup-check.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-1-discovery-neo.txt` — Full discovery output
  - [ ] `task-1-backup-check.txt` — Backup file listing

  **Commit**: NO (part of Wave 1 discovery)

- [x] 2. SSH discovery + backup on Aeza (5.182.86.27)

  **What to do**:
  - SSH to skywalker@5.182.86.27
  - Check Docker: `docker ps -a`, check for marzban-node container
  - Check Marzban node config: `ls /var/lib/marzban-node/`, check `xray_config.json`
  - Check Xray version: `docker exec marzban-node xray version 2>/dev/null`
  - Check Nginx: `nginx -t`, `ls /etc/nginx/sites-enabled/`, check for existing server blocks
  - Check port 443 usage: `ss -tlnp | grep :443` — CloudPanel should be here
  - Check port 80 usage: `ss -tlnp | grep :80`
  - Check all listening ports: `ss -tlnp`
  - Backup: `/var/lib/marzban-node/` configs, `/etc/nginx/sites-enabled/`
  - Check firewall: `ufw status`

  **Must NOT do**:
  - Do NOT touch CloudPanel or anything on port 443
  - Do NOT modify any configs
  - Do NOT restart services

  **Recommended Agent Profile**:
  - **Category**: `deep`
    - Reason: Multi-service discovery on production server with CloudPanel constraint
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed — pure SSH exploration

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1, 3, 4)
  - **Blocks**: Task 7, Task 8
  - **Blocked By**: None

  **References**:
  - `TOPOLOGY.md` — Aeza role, old IP 62.60.244.102 vs current 5.182.86.27
  - `deploy-keys.md` (Tolaria) — SSH: `ssh skywalker@5.182.86.27`
  - `OPERATIONS_AND_ROLLBACK.md` — "Do not touch: CloudPanel, sabram.ru, 443 on Aeza"

  **Acceptance Criteria**:
  - [ ] Report: Docker state, marzban-node container status, Xray version, Nginx state, port usage map (80, 443, 447, 773, 8443), firewall
  - [ ] Backup files saved to: `backups/aeza/`

  **QA Scenarios**:
  ```
  Scenario: Port conflict audit
    Tool: Bash (SSH)
    Preconditions: SSH access to skywalker@5.182.86.27
    Steps:
      1. ssh skywalker@5.182.86.27 'ss -tlnp | grep -E ":80 |:443 |:8443 "' — check target ports
      2. ssh skywalker@5.182.86.27 'ss -tlnp | grep -E ":447 |:773 "' — check old ports
    Expected Result: Port 443 occupied (CloudPanel), port 80 status reported, ports 447/773 status reported, port 8443 free
    Failure Indicators: Port 8443 already occupied → need alternative port; port 80 occupied → mask needs alternative
    Evidence: .omo/evidence/task-2-port-audit.txt

  Scenario: Backup verified
    Tool: Bash (local)
    Steps:
      1. ls backups/aeza/
      2. wc -l backups/aeza/*
    Expected Result: At least 2 config files backed up
    Evidence: .omo/evidence/task-2-backup-check.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-2-discovery-aeza.txt`
  - [ ] `task-2-port-audit.txt`
  - [ ] `task-2-backup-check.txt`

  **Commit**: NO

- [x] 3. SSH discovery on USA (184.174.97.95)

  **What to do**:
  - SSH using key: `ssh -i /Users/sabram/.ssh/skywalker_deploy skywalker@184.174.97.95`
  - Check OS: `uname -a`, `lsb_release -a`
  - Check Docker: `docker --version 2>/dev/null || echo "NO_DOCKER"`, `docker ps -a 2>/dev/null`
  - If no Docker: `which docker`, `apt list --installed | grep docker`
  - Check firewall: `ufw status`, `iptables -L -n | head -20`
  - Check listening ports: `ss -tlnp`
  - Check SSH key: `cat ~/.ssh/authorized_keys` — verify skywalker_deploy.pub is present
  - Check connectivity to Sabram Neo: `nc -zv -w3 5.42.110.191 9443` or `timeout 3 bash -c '</dev/tcp/5.42.110.191/9443' && echo OPEN || echo CLOSED`
  - Check disk space: `df -h /`
  - Check memory: `free -m`

  **Must NOT do**:
  - Do NOT install anything yet
  - Do NOT modify firewall rules

  **Recommended Agent Profile**:
  - **Category**: `deep`
    - Reason: Greenfield server assessment requiring thorough check
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1, 2, 4)
  - **Blocks**: Task 6, Task 9
  - **Blocked By**: None

  **References**:
  - `deploy-keys.md` (Tolaria) — SSH: `ssh skywalker@184.174.97.95`, key: `/Users/sabram/.ssh/skywalker_deploy.pub`
  - `DevOps/deploy-for-a.md` (Tolaria) — SSH key setup instructions
  - `TOPOLOGY.md` — USA role as 2nd data plane

  **Acceptance Criteria**:
  - [ ] Report: OS version, Docker state (installed/missing), firewall status, port 2096 availability, panel connectivity (can reach 5.42.110.191:9443), disk/memory
  - [ ] SSH key confirmed working

  **QA Scenarios**:
  ```
  Scenario: SSH connectivity works
    Tool: Bash (SSH)
    Preconditions: skywalker_deploy key exists at /Users/sabram/.ssh/skywalker_deploy
    Steps:
      1. ssh -i /Users/sabram/.ssh/skywalker_deploy -o ConnectTimeout=5 skywalker@184.174.97.95 'echo OK'
    Expected Result: "OK" returned within 5 seconds
    Failure Indicators: Connection timeout, permission denied → SSH key not deployed
    Evidence: .omo/evidence/task-3-ssh-check.txt

  Scenario: Panel reachability from USA
    Tool: Bash (SSH)
    Steps:
      1. ssh -i /Users/sabram/.ssh/skywalker_deploy skywalker@184.174.97.95 'timeout 3 bash -c "</dev/tcp/5.42.110.191/9443" && echo REACHABLE || echo UNREACHABLE'
    Expected Result: REACHABLE — USA can connect to Sabram Neo panel port
    Failure Indicators: UNREACHABLE — firewall on Sabram Neo blocks USA → need to open port 9443
    Evidence: .omo/evidence/task-3-panel-reachability.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-3-discovery-usa.txt`
  - [ ] `task-3-ssh-check.txt`
  - [ ] `task-3-panel-reachability.txt`

  **Commit**: NO

- [x] 4. Git init + initial commit of current state

  **What to do**:
  - `git init` in project root
  - Create `.gitignore`:
    ```
    .env
    *.key
    *.pem
    *.crt
    backups/
    Archive/
    .omo/evidence/
    .DS_Store
    ```
  - Stage all current docs (README.md, TOPOLOGY.md, CURRENT_STATE.md, ACTIVE_SUBSCRIPTIONS.md, OPERATIONS_AND_ROLLBACK.md, TODO.md, SOURCE_OF_TRUTH.env, XRAY_CONFIG.current.json, AGENTS.md)
  - Make initial commit: `git add -A && git commit -m "chore: initial state — dead VPN before reanimation"`
  - Create branch: `git checkout -b "Rise of Republic"`
  - Note remote URL: `git@github.com:Sabramvi/Rebel-Alliance.git`

  **Must NOT do**:
  - Do NOT commit secrets (check .env, keys, certs are gitignored)
  - Do NOT commit backups/ or Archive/
  - Do NOT push yet (will push in Task 27)

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Simple git operations, no complexity
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - `git-master`: Not needed — operations are trivial (init, add, commit, branch)

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1, 2, 3)
  - **Blocks**: None (foundational, but no task depends on git state)
  - **Blocked By**: None

  **References**:
  - `AGENTS.md` — Git repo URL: `git@github.com:Sabramvi/Rebel-Alliance.git`
  - `TODO.md` — "Init the GIT repo. Make 1st commit of initial state. Then make new branch named Rise of Republic"

  **Acceptance Criteria**:
  - [ ] `git log --oneline` shows 1 commit on `Rise of Republic` branch
  - [ ] `.gitignore` exists with secrets/backups/Archive patterns
  - [ ] `git status` is clean (no untracked secrets)

  **QA Scenarios**:
  ```
  Scenario: Git repo initialized correctly
    Tool: Bash (local)
    Steps:
      1. git branch --show-current — should be "Rise of Republic"
      2. git log --oneline — should show exactly 1 commit
      3. cat .gitignore — verify contains .env, backups/, Archive/, .omo/evidence/
      4. git status — should be clean
    Expected Result: Branch "Rise of Republic", 1 commit, .gitignore present, clean status
    Evidence: .omo/evidence/task-4-git-init.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-4-git-init.txt`

  **Commit**: YES (groups with this task)
  - Message: `chore: init git repo — dead VPN state before reanimation`
  - Files: `.gitignore`, all current docs
  - Pre-commit: verify no secrets staged

- [x] 5. Fix Marzban panel on Sabram Neo (5.42.110.191)

  **What to do**:
  - Based on Task 1 discovery, fix the broken panel
  - Common fixes (choose based on discovery):
    - If Docker container stopped/restarting: `docker restart marzban-marzban-1`, check logs
    - If Nginx misconfigured: fix proxy_pass to point to Marzban container port, set server_name to mandalore.severdesign.ru
    - If SSL cert expired/missing: use self-signed cert temporarily (real LE cert comes in Task 10)
    - If DB corrupted: attempt `sqlite3 /var/lib/marzban/db.sqlite3 ".dump" | sqlite3 /var/lib/marzban/db.sqlite3.new`
    - If .env missing: restore from backup or create with minimal config
  - Configure Nginx to serve panel on port 9443:
    ```nginx
    server {
        listen 9443 ssl;
        server_name mandalore.severdesign.ru;
        ssl_certificate /etc/letsencrypt/live/mandalore.severdesign.ru/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/mandalore.severdesign.ru/privkey.pem;
        location / {
            proxy_pass http://127.0.0.1:8000;  # or whatever Marzban internal port is
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
    ```
  - Ensure Marzban .env has correct settings: UVICORN_HOST=0.0.0.0, UVICORN_PORT=8000, XRAY_SUBSCRIPTION_URL_PREFIX=https://mandalore.severdesign.ru:9443
  - Restart services: `docker restart marzban-marzban-1`, `nginx -s reload`
  - Verify panel loads: `curl -sk https://localhost:9443/dashboard/login`

  **Must NOT do**:
  - Do NOT reinstall Marzban
  - Do NOT delete database
  - Do NOT change Marzban admin credentials

  **Recommended Agent Profile**:
  - **Category**: `deep`
    - Reason: Debugging broken production service requires careful diagnosis and minimal intervention
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2 (with Tasks 6, 7, 8; Task 9 depends on 6)
  - **Blocks**: Task 10
  - **Blocked By**: Task 1

  **References**:
  - Task 1 discovery output — Current state of Marzban
  - `OPERATIONS_AND_ROLLBACK.md` — Rollback steps (restore DB snapshot, restart container)
  - `SOURCE_OF_TRUTH.env` — Old env mapping for reference

  **Acceptance Criteria**:
  - [ ] `docker ps` shows marzban-marzban-1 running and healthy
  - [ ] `curl -sk https://127.0.0.1:9443/dashboard/login` returns 200 with login form
  - [ ] `curl -sk https://5.42.110.191:9443/dashboard/login` returns 200 (external accessible)
  - [ ] Nginx config passes `nginx -t`

  **QA Scenarios**:
  ```
  Scenario: Panel responds on port 9443
    Tool: Bash (SSH + curl)
    Preconditions: Marzban container running, Nginx configured
    Steps:
      1. ssh skywalker@5.42.110.191 'curl -sk -o /dev/null -w "%{http_code}" https://127.0.0.1:9443/dashboard/login'
    Expected Result: 200
    Failure Indicators: 502 (bad gateway — proxy_pass wrong), 000 (connection refused — service not running)
    Evidence: .omo/evidence/task-5-panel-local.txt

  Scenario: Panel login page contains expected content
    Tool: Bash (SSH + curl)
    Steps:
      1. ssh skywalker@5.42.110.191 'curl -sk https://127.0.0.1:9443/dashboard/login | grep -o "<title>[^<]*</title>"'
    Expected Result: <title> contains "Marzban" or login-related text
    Evidence: .omo/evidence/task-5-panel-content.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-5-panel-local.txt`
  - [ ] `task-5-panel-content.txt`

  **Commit**: NO

- [x] 6. Install Docker + Marzban-node on USA (184.174.97.95)

  **What to do**:
  - SSH to USA with skywalker_deploy key
  - Install Docker if missing:
    ```bash
    sudo apt update && sudo apt install -y docker.io docker-compose-v2
    sudo systemctl enable --now docker
    sudo usermod -aG docker skywalker
    ```
  - Install Marzban-node:
    ```bash
    sudo mkdir -p /var/lib/marzban-node
    cd /var/lib/marzban-node
    sudo wget -O docker-compose.yml https://raw.githubusercontent.com/Gozargah/Marzban-node/master/docker-compose.yml
    ```
  - Configure node env: `/var/lib/marzban-node/.env`:
    ```
    SERVICE_PORT=2096
    XRAY_API_PORT=10085
    SSL_KEY_FILE=/var/lib/marzban-node/tatooine_key.pem
    SSL_CERT_FILE=/var/lib/marzban-node/tatooine_cert.pem
    ```
  - Do NOT start the node yet (certs not ready, panel not configured)
  - Open firewall for port 2096: `sudo ufw allow 2096/tcp`
  - Ensure node can reach panel: test connectivity to 5.42.110.191:9443

  **Must NOT do**:
  - Do NOT start marzban-node container yet
  - Do NOT open unnecessary ports (only 2096 needed for Tatooine)
  - Do NOT install CloudPanel or any other services

  **Recommended Agent Profile**:
  - **Category**: `deep`
    - Reason: Greenfield server setup with Docker, firewall, and Marzban-node configuration
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2 (with Tasks 5, 7, 8)
  - **Blocks**: Task 9, Task 12, Task 14
  - **Blocked By**: Task 3

  **References**:
  - `deploy-keys.md` (Tolaria) — SSH command and key path
  - Marzban-node docs: `https://github.com/Gozargah/Marzban-node` — docker-compose.yml URL

  **Acceptance Criteria**:
  - [ ] `docker --version` returns version string
  - [ ] `docker compose version` returns version string
  - [ ] `/var/lib/marzban-node/docker-compose.yml` exists
  - [ ] `/var/lib/marzban-node/.env` has correct SERVICE_PORT=2096
  - [ ] `sudo ufw status | grep 2096` shows port open
  - [ ] Panel reachability confirmed (from Task 3 or re-test)

  **QA Scenarios**:
  ```
  Scenario: Docker installed and running
    Tool: Bash (SSH)
    Preconditions: SSH to skywalker@184.174.97.95 with deploy key
    Steps:
      1. ssh -i /Users/sabram/.ssh/skywalker_deploy skywalker@184.174.97.95 'docker --version && docker compose version'
    Expected Result: Both return version strings (e.g., "Docker version 27.x.x")
    Failure Indicators: "command not found" → Docker not installed
    Evidence: .omo/evidence/task-6-docker-check.txt

  Scenario: Marzban-node directory prepared
    Tool: Bash (SSH)
    Steps:
      1. ssh -i /Users/sabram/.ssh/skywalker_deploy skywalker@184.174.97.95 'ls -la /var/lib/marzban-node/docker-compose.yml'
      2. ssh -i /Users/sabram/.ssh/skywalker_deploy skywalker@184.174.97.95 'cat /var/lib/marzban-node/.env | grep SERVICE_PORT'
    Expected Result: docker-compose.yml exists, .env has SERVICE_PORT=2096
    Evidence: .omo/evidence/task-6-marzban-node-prep.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-6-docker-check.txt`
  - [ ] `task-6-marzban-node-prep.txt`

  **Commit**: NO

- [x] 7. Configure Aeza Marzban-node for Coruscant (GRPC REALITY)

  **What to do**:
  - Based on Task 2 discovery, reconfigure Aeza node
  - Stop old inbounds if running: remove port 773 (Mos Eisley) and 447 (Alderaan) configurations
  - Edit `/var/lib/marzban-node/xray_config.json` or Marzban node .env:
    - Add Coruscant inbound: VLESS GRPC REALITY on port 8443
    - serviceName: `cor-grpc`
    - serverName: coruscant.severdesign.ru
    - Set reality dest: coruscant.severdesign.ru:8443 (will use LE cert from Task 11)
  - Open firewall: `sudo ufw allow 8443/tcp`
  - Ensure no conflict with port 443 (CloudPanel)
  - Restart node container: `docker restart marzban-node`
  - Verify: `ss -tlnp | grep 8443` shows listening

  **Must NOT do**:
  - Do NOT touch port 443 (CloudPanel)
  - Do NOT remove old configs without backup (backup done in Task 2)
  - Do NOT break existing node-panel connection if it exists

  **Recommended Agent Profile**:
  - **Category**: `deep`
    - Reason: Reconfiguring production Marzban-node with GRPC REALITY — complex protocol setup
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2 (with Tasks 5, 6, 8)
  - **Blocks**: Task 11
  - **Blocked By**: Task 2

  **References**:
  - Task 2 discovery output — Current Aeza node state
  - `XRAY_CONFIG.current.json` — Old config for structure reference
  - `OPERATIONS_AND_ROLLBACK.md` — "restart the node container on Aeza only if the listener still fails"

  **Acceptance Criteria**:
  - [ ] `sudo ufw status | grep 8443` shows port open
  - [ ] `ss -tlnp | grep 8443` shows listening (after node restart)
  - [ ] Old ports 773 and 447 NOT listening
  - [ ] Port 443 still occupied by CloudPanel (unchanged)

  **QA Scenarios**:
  ```
  Scenario: Coruscant port 8443 listening
    Tool: Bash (SSH)
    Preconditions: Aeza node reconfigured, container restarted
    Steps:
      1. ssh skywalker@5.182.86.27 'ss -tlnp | grep ":8443 "'
    Expected Result: Line showing port 8443 listening (xray or marzban-node process)
    Failure Indicators: No output → port not listening, node not started, config error
    Evidence: .omo/evidence/task-7-port-listen.txt

  Scenario: Old ports cleaned up
    Tool: Bash (SSH)
    Steps:
      1. ssh skywalker@5.182.86.27 'ss -tlnp | grep -E ":773 |:447 "'
    Expected Result: No output (neither old port listening)
    Evidence: .omo/evidence/task-7-old-ports.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-7-port-listen.txt`
  - [ ] `task-7-old-ports.txt`

  **Commit**: NO

- [x] 8. Nginx mask: jedha.severdesign.ru → hello.severdesign.ru on Aeza

  **What to do**:
  - Add Nginx server block on Aeza for jedha mask:
    ```nginx
    server {
        listen 80;
        server_name jedha.severdesign.ru;
        return 301 http://hello.severdesign.ru;
    }
    ```
  - Place in `/etc/nginx/sites-enabled/jedha-mask`
  - Test: `nginx -t`
  - Reload: `nginx -s reload`
  - Verify: `curl -sI http://jedha.severdesign.ru` → 301 to hello.severdesign.ru

  **Must NOT do**:
  - Do NOT touch existing CloudPanel Nginx configs
  - Do NOT modify port 443 server blocks
  - Do NOT create duplicate server_name entries

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Simple Nginx redirect — single file, straightforward
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2 (with Tasks 5, 6, 7)
  - **Blocks**: Task 11, Task 13
  - **Blocked By**: Task 2

  **References**:
  - Task 2 discovery output — Current Nginx state on Aeza
  - `TOPOLOGY.md` — jedha.severdesign.ru role as TLS mask

  **Acceptance Criteria**:
  - [ ] `nginx -t` passes
  - [ ] `curl -sI http://jedha.severdesign.ru` → 301 Location: http://hello.severdesign.ru
  - [ ] CloudPanel still works on port 443 (verified)

  **QA Scenarios**:
  ```
  Scenario: Jedha redirects to hello
    Tool: Bash (curl from local machine — DNS resolves jedha to Aeza)
    Preconditions: Nginx reloaded, DNS for jedha.severdesign.ru → 5.182.86.27
    Steps:
      1. curl -sI --max-time 5 http://jedha.severdesign.ru | grep -E "HTTP|Location"
    Expected Result: HTTP/1.1 301, Location: http://hello.severdesign.ru
    Failure Indicators: Connection refused, timeout, wrong Location header
    Evidence: .omo/evidence/task-8-jedha-redirect.txt

  Scenario: CloudPanel unaffected
    Tool: Bash (SSH)
    Steps:
      1. ssh skywalker@5.182.86.27 'curl -sk -o /dev/null -w "%{http_code}" https://127.0.0.1:443'
    Expected Result: 200 or 302 (CloudPanel responding)
    Evidence: .omo/evidence/task-8-cloudpanel-ok.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-8-jedha-redirect.txt`
  - [ ] `task-8-cloudpanel-ok.txt`

  **Commit**: NO

- [x] 9. Nginx mask: bespin.severdesign.ru → hello.severdesign.ru on USA

  **What to do**:
  - Install Nginx on USA if not present: `sudo apt install -y nginx`
  - Add server block:
    ```nginx
    server {
        listen 80;
        server_name bespin.severdesign.ru;
        return 301 http://hello.severdesign.ru;
    }
    ```
  - Enable, test, reload
  - Open port 80: `sudo ufw allow 80/tcp`
  - Verify: `curl -sI http://bespin.severdesign.ru` → 301

  **Must NOT do**:
  - Do NOT install additional services beyond Nginx
  - Do NOT configure HTTPS on USA yet (cert comes in Task 14)

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Simple Nginx install + redirect config on clean server
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: NO (depends on Task 6)
  - **Parallel Group**: Wave 2 — runs after Task 6 completes
  - **Blocks**: Task 12, Task 14
  - **Blocked By**: Task 3, Task 6

  **References**:
  - Task 3 discovery output — USA server state
  - `TOPOLOGY.md` — bespin.severdesign.ru as TLS mask for USA

  **Acceptance Criteria**:
  - [ ] `nginx -t` passes
  - [ ] `curl -sI http://bespin.severdesign.ru` → 301 Location: http://hello.severdesign.ru
  - [ ] `sudo ufw status | grep 80` shows port open

  **QA Scenarios**:
  ```
  Scenario: Bespin redirects to hello
    Tool: Bash (curl from local — DNS resolves bespin to USA)
    Preconditions: Nginx running, DNS bespin.severdesign.ru → 184.174.97.95
    Steps:
      1. curl -sI --max-time 5 http://bespin.severdesign.ru | grep -E "HTTP|Location"
    Expected Result: HTTP/1.1 301, Location: http://hello.severdesign.ru
    Failure Indicators: Connection refused, timeout, DNS not resolving to USA
    Evidence: .omo/evidence/task-9-bespin-redirect.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-9-bespin-redirect.txt`

  **Commit**: NO

- [x] 10. Let's Encrypt DNS cert for mandalore.severdesign.ru (Sabram Neo)

  **What to do**:
  - Install certbot with DNS plugin: `sudo apt install -y certbot python3-certbot-dns-cloudflare` (or appropriate DNS provider plugin)
  - Obtain API token for DNS provider from Tolaria vault (Cloudflare/Namecheap — discover which provider)
  - Create credentials file for DNS challenge
  - Run: `sudo certbot certonly --dns-cloudflare --dns-cloudflare-credentials /root/.secrets/cloudflare.ini -d mandalore.severdesign.ru`
  - Test cert: `openssl x509 -in /etc/letsencrypt/live/mandalore.severdesign.ru/fullchain.pem -noout -subject -dates`
  - Update Nginx to use the real cert (replace temp self-signed from Task 5)
  - Reload Nginx: `nginx -s reload`
  - Setup auto-renewal: certbot renew timer should be active by default

  **Must NOT do**:
  - Do NOT commit API tokens to git
  - Do NOT overwrite CloudPanel certs (if any exist on Sabram Neo)

  **Recommended Agent Profile**:
  - **Category**: `deep`
    - Reason: DNS-based Let's Encrypt with provider API token — requires careful credential handling
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 3 (with Tasks 11, 12, 13, 14)
  - **Blocks**: Task 15, Task 16
  - **Blocked By**: Task 5

  **References**:
  - `TOPOLOGY.md` — mandalore.severdesign.ru as panel domain
  - Tolaria vault — DNS provider API tokens if stored there
  - certbot docs: `https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal`

  **Acceptance Criteria**:
  - [ ] `/etc/letsencrypt/live/mandalore.severdesign.ru/fullchain.pem` exists
  - [ ] `openssl x509 -noout -subject` shows CN=mandalore.severdesign.ru
  - [ ] Cert expiry > 80 days from now
  - [ ] `curl -sk https://mandalore.severdesign.ru:9443/dashboard/login` returns 200 with valid cert (no --insecure needed from trusted machine)

  **QA Scenarios**:
  ```
  Scenario: Valid TLS cert from external perspective
    Tool: Bash (local machine — DNS resolves mandalore to 5.42.110.191)
    Preconditions: DNS mandalore.severdesign.ru → 5.42.110.191
    Steps:
      1. echo | openssl s_client -connect mandalore.severdesign.ru:9443 -servername mandalore.severdesign.ru 2>/dev/null | openssl x509 -noout -subject -dates
    Expected Result: subject=CN=mandalore.severdesign.ru, notBefore in past, notAfter >80 days from now
    Failure Indicators: self-signed cert, wrong CN, expired
    Evidence: .omo/evidence/task-10-mandalore-cert.txt

  Scenario: Panel accessible with HTTPS
    Tool: Bash (curl)
    Steps:
      1. curl -sk -o /dev/null -w "%{http_code}" https://mandalore.severdesign.ru:9443/dashboard/login
    Expected Result: 200
    Evidence: .omo/evidence/task-10-mandalore-https.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-10-mandalore-cert.txt`
  - [ ] `task-10-mandalore-https.txt`

  **Commit**: NO

- [x] 11. Let's Encrypt DNS cert for coruscant.severdesign.ru (Aeza)

  **What to do**:
  - Similar to Task 10 but on Aeza server
  - Install certbot + DNS plugin on Aeza
  - Run: `sudo certbot certonly --dns-cloudflare ... -d coruscant.severdesign.ru`
  - Copy cert to Marzban-node cert path: `/var/lib/marzban-node/`
  - Update Marzban-node `.env` or config to point to cert
  - Restart marzban-node container: `docker restart marzban-node`

  **Must NOT do**:
  - Do NOT touch CloudPanel certs
  - Do NOT modify port 443 Nginx config

  **Recommended Agent Profile**:
  - **Category**: `deep`
    - Reason: LE cert on production server with CloudPanel coexistence constraint
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 3 (with Tasks 10, 12, 13, 14)
  - **Blocks**: Task 15
  - **Blocked By**: Task 7, Task 8

  **References**:
  - Task 7 output — Marzban-node config paths on Aeza
  - `TOPOLOGY.md` — coruscant.severdesign.ru as edge domain

  **Acceptance Criteria**:
  - [ ] `/etc/letsencrypt/live/coruscant.severdesign.ru/fullchain.pem` exists
  - [ ] Cert CN = coruscant.severdesign.ru
  - [ ] Marzban-node restarted and using new cert
  - [ ] `openssl s_client -connect coruscant.severdesign.ru:8443 -servername coruscant.severdesign.ru` shows valid cert

  **QA Scenarios**:
  ```
  Scenario: Coruscant REALITY TLS cert valid
    Tool: Bash (local — DNS resolves coruscant to 5.182.86.27)
    Preconditions: Marzban-node restarted with cert
    Steps:
      1. echo | openssl s_client -connect coruscant.severdesign.ru:8443 -servername coruscant.severdesign.ru 2>/dev/null | openssl x509 -noout -subject -dates
    Expected Result: subject=CN=coruscant.severdesign.ru, cert valid
    Failure Indicators: self-signed, wrong CN, cert not found
    Evidence: .omo/evidence/task-11-coruscant-cert.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-11-coruscant-cert.txt`

  **Commit**: NO

- [x] 12. Let's Encrypt DNS certs for tatooine.severdesign.ru + dagoba.severdesign.ru (USA)

  **What to do**:
  - Install certbot + DNS plugin on USA
  - Obtain cert for BOTH domains: `sudo certbot certonly --dns-cloudflare ... -d tatooine.severdesign.ru -d dagoba.severdesign.ru`
  - Copy cert to `/var/lib/marzban-node/`:
    ```bash
    sudo cp /etc/letsencrypt/live/tatooine.severdesign.ru/fullchain.pem /var/lib/marzban-node/tatooine_cert.pem
    sudo cp /etc/letsencrypt/live/tatooine.severdesign.ru/privkey.pem /var/lib/marzban-node/tatooine_key.pem
    sudo chown skywalker:skywalker /var/lib/marzban-node/*.pem
    ```
  - Start Marzban-node for first time: `cd /var/lib/marzban-node && docker compose up -d`
  - Verify: `docker ps | grep marzban-node`, `ss -tlnp | grep 2096`

  **Must NOT do**:
  - Do NOT start node without certs in place
  - Do NOT expose port 80 or 443 for HTTP challenge (use DNS only)

  **Recommended Agent Profile**:
  - **Category**: `deep`
    - Reason: Dual-cert LE + first node startup on greenfield server
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 3 (with Tasks 10, 11, 13, 14)
  - **Blocks**: Task 16
  - **Blocked By**: Task 6, Task 9

  **References**:
  - Task 6 output — Marzban-node directory on USA
  - `TOPOLOGY.md` — tatooine as edge, dagoba as SNI

  **Acceptance Criteria**:
  - [ ] `/etc/letsencrypt/live/tatooine.severdesign.ru/fullchain.pem` exists with both SANs
  - [ ] `/var/lib/marzban-node/tatooine_cert.pem` and `tatooine_key.pem` exist
  - [ ] Marzban-node container running: `docker ps | grep marzban-node`
  - [ ] Port 2096 listening: `ss -tlnp | grep 2096`
  - [ ] `openssl s_client -connect tatooine.severdesign.ru:2096 -servername dagoba.severdesign.ru | grep "Verify return code: 0"`

  **QA Scenarios**:
  ```
  Scenario: Tatooine XHTTP TLS handshake with dagoba SNI
    Tool: Bash (local — DNS tatooine → 184.174.97.95)
    Preconditions: Marzban-node running with certs
    Steps:
      1. echo | openssl s_client -connect tatooine.severdesign.ru:2096 -servername dagoba.severdesign.ru 2>/dev/null | openssl x509 -noout -subject -dates
    Expected Result: subject includes dagoba.severdesign.ru or tatooine.severdesign.ru, cert valid
    Failure Indicators: SSL error, wrong SNI response, connection refused
    Evidence: .omo/evidence/task-12-tatooine-cert.txt

  Scenario: USA node running
    Tool: Bash (SSH)
    Steps:
      1. ssh -i /Users/sabram/.ssh/skywalker_deploy skywalker@184.174.97.95 'docker ps --format "{{.Names}} {{.Status}}" | grep marzban'
    Expected Result: marzban-node container with "Up" status
    Evidence: .omo/evidence/task-12-usa-node-status.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-12-tatooine-cert.txt`
  - [ ] `task-12-usa-node-status.txt`

  **Commit**: NO

- [x] 13. Let's Encrypt DNS cert for jedha.severdesign.ru (Aeza)

  **What to do**:
  - Obtain cert on Aeza: `sudo certbot certonly --dns-cloudflare ... -d jedha.severdesign.ru`
  - Update Nginx jedha server block to also listen on 443 with SSL (optional — HTTP redirect is minimum viable)
  - If adding HTTPS:
    ```nginx
    server {
        listen 443 ssl;
        server_name jedha.severdesign.ru;
        ssl_certificate /etc/letsencrypt/live/jedha.severdesign.ru/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/jedha.severdesign.ru/privkey.pem;
        return 301 https://hello.severdesign.ru;
    }
    ```
  - Reload Nginx

  **Must NOT do**:
  - Do NOT conflict with CloudPanel's port 443 — check if 443 has existing server block for jedha
  - If 443 conflict, keep jedha HTTP-only on port 80

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Simple LE cert + optional Nginx update
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 3 (with Tasks 10, 11, 12, 14)
  - **Blocks**: Task 23 (QA)
  - **Blocked By**: Task 8

  **References**:
  - Task 8 output — jedha Nginx config

  **Acceptance Criteria**:
  - [ ] `/etc/letsencrypt/live/jedha.severdesign.ru/fullchain.pem` exists
  - [ ] Cert CN = jedha.severdesign.ru
  - [ ] Nginx reloaded without errors
  - [ ] HTTP redirect still works

  **QA Scenarios**:
  ```
  Scenario: Jedha HTTP redirect still works after cert install
    Tool: Bash (local)
    Steps:
      1. curl -sI --max-time 5 http://jedha.severdesign.ru | grep "Location"
    Expected Result: Location: http://hello.severdesign.ru
    Evidence: .omo/evidence/task-13-jedha-redirect.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-13-jedha-redirect.txt`

  **Commit**: NO

- [x] 14. Let's Encrypt DNS cert for bespin.severdesign.ru (USA)

  **What to do**:
  - Similar to Task 13 but on USA
  - Obtain cert: `sudo certbot certonly --dns-cloudflare ... -d bespin.severdesign.ru`
  - Optionally add HTTPS redirect on 443
  - Reload Nginx

  **Must NOT do**:
  - Do NOT conflict with Tatooine on port 2096 (different ports, safe)

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Simple LE cert on USA
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 3 (with Tasks 10, 11, 12, 13)
  - **Blocks**: Task 23 (QA)
  - **Blocked By**: Task 6, Task 9

  **References**:
  - Task 9 output — bespin Nginx config on USA

  **Acceptance Criteria**:
  - [ ] `/etc/letsencrypt/live/bespin.severdesign.ru/fullchain.pem` exists
  - [ ] Nginx reloaded without errors

  **QA Scenarios**:
  ```
  Scenario: Bespin HTTP redirect still works
    Tool: Bash (local)
    Steps:
      1. curl -sI --max-time 5 http://bespin.severdesign.ru | grep "Location"
    Expected Result: Location: http://hello.severdesign.ru
    Evidence: .omo/evidence/task-14-bespin-redirect.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-14-bespin-redirect.txt`

  **Commit**: NO

- [x] 15. Configure Coruscant inbound on Marzban panel

  **What to do**:
  - Access Marzban panel at `https://mandalore.severdesign.ru:9443/dashboard/`
  - Navigate to Inbounds → Add new inbound
  - Configure:
    - Tag: `Coruscant`
    - Protocol: VLESS
    - Network: GRPC
    - Security: REALITY
    - Port: 8443
    - Address: coruscant.severdesign.ru
    - serviceName: `cor-grpc`
    - Node: Aeza (5.182.86.27)
    - Enable: YES
  - Set REALITY fallback dest: `coruscant.severdesign.ru:8443` (self-fallback with valid TLS cert makes it more stealthy)
  - Set REALITY private key (generated or from existing)
  - Set REALITY shortIds
  - If GUI inaccessible, configure directly via Marzban API or edit panel DB/JSON
  - Verify: Inbounds list shows Coruscant, status "active"

  **Must NOT do**:
  - Do NOT delete old inbounds yet (Task 17 handles removal)
  - Do NOT change existing node settings

  **Recommended Agent Profile**:
  - **Category**: `deep`
    - Reason: REALITY inbound configuration is complex — requires understanding of VLESS+GRPC+REALITY parameters
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 4 (with Task 16; Tasks 17, 18, 19 depend on 15 AND 16)
  - **Blocks**: Task 17, Task 18, Task 19
  - **Blocked By**: Task 10, Task 11

  **References**:
  - `XRAY_CONFIG.current.json` — Old REALITY inbound structure for reference (Alderaan)
  - Marzban docs: Inbound configuration
  - `TOPOLOGY.md` — Coruscant as GRPC REALITY edge

  **Acceptance Criteria**:
  - [ ] Marzban panel shows Coruscant inbound in list
  - [ ] Inbound status: active/connected
  - [ ] Aeza node shows as connected in panel Nodes section

  **QA Scenarios**:
  ```
  Scenario: Coruscant inbound visible in panel
    Tool: Bash (curl Marzban API via SSH to Sabram Neo)
    Preconditions: Panel admin session cookie obtained
    Steps:
      1. ssh skywalker@5.42.110.191 'curl -sk -b /tmp/marzban_cookie https://127.0.0.1:8000/api/inbounds | python3 -m json.tool | grep -A5 Coruscant'
    Expected Result: JSON block with tag "Coruscant", port 8443, protocol "vless"
    Failure Indicators: No Coruscant in response → inbound not created
    Evidence: .omo/evidence/task-15-coruscant-api.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-15-coruscant-api.txt`

  **Commit**: NO

- [x] 16. Configure Tatooine inbound on Marzban panel

  **What to do**:
  - Access Marzban panel → Inbounds → Add new inbound
  - Configure:
    - Tag: `Tatooine`
    - Protocol: VLESS
    - Network: XHTTP
    - Security: TLS
    - Port: 2096
    - Address: tatooine.severdesign.ru
    - SNI: dagoba.severdesign.ru
    - Path: `/api`
    - Node: USA (184.174.97.95)
    - Enable: YES
  - Set xmux parameters from Atlanta profile:
    - cMaxReuseTimes: 64-128
    - hMaxRequestTimes: 600-900
    - maxConcurrency: 16-32
    - xPaddingBytes: 100-1000
  - Verify: Inbounds list shows Tatooine, status "active"

  **Must NOT do**:
  - Do NOT delete old inbounds yet

  **Recommended Agent Profile**:
  - **Category**: `deep`
    - Reason: XHTTP TLS inbound with xmux parameters — complex configuration
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 4 (with Task 15)
  - **Blocks**: Task 17, Task 18, Task 19
  - **Blocked By**: Task 10, Task 12

  **References**:
  - `Atlanta-xhttp-profile-june.json` — xmux parameters (cMaxReuseTimes, hMaxRequestTimes, maxConcurrency, xPaddingBytes)
  - `XRAY_CONFIG.current.json` — Old XHTTP inbound structure for reference
  - `TOPOLOGY.md` — Tatooine as XHTTP TLS edge, dagoba as SNI

  **Acceptance Criteria**:
  - [ ] Marzban panel shows Tatooine inbound in list
  - [ ] Inbound status: active/connected
  - [ ] USA node shows as connected in panel Nodes section

  **QA Scenarios**:
  ```
  Scenario: Tatooine inbound visible in panel
    Tool: Bash (SSH to Sabram Neo)
    Steps:
      1. ssh skywalker@5.42.110.191 'curl -sk https://127.0.0.1:8000/api/inbounds | python3 -m json.tool | grep -A5 Tatooine'
    Expected Result: JSON block with tag "Tatooine", port 2096, SNI dagoba
    Evidence: .omo/evidence/task-16-tatooine-api.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-16-tatooine-api.txt`

  **Commit**: NO

- [x] 17. Remove old inbounds + old users from Marzban panel

  **What to do**:
  - Access Marzban panel → Inbounds
  - Delete: Mos Eisley (xhttp+tls, port 773) and Alderaan (grpc+reality, port 447)
  - Verify old ports NOT in inbound list
  - Remove old users: Sabram, AnnasSweets, MadameSabram, Siv34 (preserve their data in backup)
  - Verify only 1 user remains → Anakin (will be created in Task 19 if not exists)
  - On Aeza: ensure old port listeners are stopped (`docker restart marzban-node` if needed)

  **Must NOT do**:
  - Do NOT delete without verifying backups exist (from Task 1, 2)
  - Do NOT delete the admin/panel user

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Simple deletion operations in panel UI/API
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: NO (depends on 15, 16; runs after both)
  - **Parallel Group**: Wave 4 (after Tasks 15, 16)
  - **Blocks**: None
  - **Blocked By**: Task 15, Task 16

  **References**:
  - `ACTIVE_SUBSCRIPTIONS.md` — Old user list (Sabram, AnnasSweets, MadameSabram, Siv34)
  - `TODO.md` — "only 1 user Anakin"

  **Acceptance Criteria**:
  - [ ] Panel inbounds: only Coruscant + Tatooine (2 total)
  - [ ] Panel users: only Anakin + admin (2 total)
  - [ ] `ss -tlnp` on Aeza: ports 773, 447 NOT listening

  **QA Scenarios**:
  ```
  Scenario: Only 2 inbounds exist
    Tool: Bash (SSH to Sabram Neo → Marzban API)
    Steps:
      1. ssh skywalker@5.42.110.191 'curl -sk https://127.0.0.1:8000/api/inbounds | python3 -c "import sys,json; data=json.load(sys.stdin); print(len(data)); [print(i[\"tag\"]) for i in data]"'
    Expected Result: 2 inbounds, tags: Coruscant, Tatooine
    Evidence: .omo/evidence/task-17-inbounds-count.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-17-inbounds-count.txt`

  **Commit**: NO

- [x] 18. Configure routing rules on Marzban panel

  **What to do**:
  - Configure Marzban Hosts/Routing settings:
  - **PROXY (force through VPN)**:
    - `adobe.com`, `adobe.io`, `adobedtm.com`
    - `openai.com`, `oaistatic.com`, `oaiusercontent.com`
    - `anthropic.com`, `claude.ai`
    - `gemini.google.com`, `generativelanguage.googleapis.com`, `ai.google.dev`
    - `antigravity.dev`, `antigravity.google` (if applicable)
  - **DIRECT (bypass VPN, from Atlanta template)**:
    - `*.ru`, `*.su`, `*.by`, `*.xn--p1ai`
    - `push.apple.com`, `api.push.apple.com`
    - Bittorrent protocol
    - IP range: `17.0.0.0/8` (Apple)
    - All RU domains from Atlanta-xhttp-profile-june.json routing rules
  - Apply to both inbounds (Coruscant + Tatooine)
  - Verify rules are saved and show in panel routing section

  **Must NOT do**:
  - Do NOT add routing rules beyond what's specified (no Netflix, YouTube, etc.)
  - Do NOT block traffic — only route

  **Recommended Agent Profile**:
  - **Category**: `deep`
    - Reason: Complex routing rules with large domain lists — requires careful configuration
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: NO (depends on 15, 16; runs after Task 17)
  - **Parallel Group**: Wave 4 (after Tasks 15, 16, 17)
  - **Blocks**: Task 24 (QA routing)
  - **Blocked By**: Task 15, Task 16

  **References**:
  - `Atlanta-xhttp-profile-june.json` — Complete routing rules (lines 134-468)
  - `TODO.md` — "strict settings for all adobe, openai, anthropic, gemini, antigravity sites and services"

  **Acceptance Criteria**:
  - [ ] Routing section in Marzban panel shows rules for AI services → proxy
  - [ ] Routing section shows RU domains → direct
  - [ ] Both inbounds have routing rules applied

  **QA Scenarios**:
  ```
  Scenario: AI domains route through proxy
    Tool: Bash (SSH to Sabram Neo → check Marzban routing config)
    Steps:
      1. ssh skywalker@5.42.110.191 'grep -r "openai\|anthropic\|gemini\|adobe\|antigravity" /var/lib/marzban/ 2>/dev/null | head -10'
    Expected Result: grep finds routing entries for these domains in Marzban config
    Evidence: .omo/evidence/task-18-ai-routing.txt

  Scenario: RU domains route direct
    Tool: Bash (SSH)
    Steps:
      1. ssh skywalker@5.42.110.191 'grep -r "vk.com\|yandex\|mail.ru" /var/lib/marzban/ 2>/dev/null | head -10'
    Expected Result: grep finds routing entries with "direct" for these domains
    Evidence: .omo/evidence/task-18-ru-routing.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-18-ai-routing.txt`
  - [ ] `task-18-ru-routing.txt`

  **Commit**: NO

- [x] 19. Create user "Anakin" + subscription "Rebel Alliance"

  **What to do**:
  - Access Marzban panel → Users → Add User
  - Username: `Anakin`
  - Status: Active
  - Data limit: reasonable (e.g., 500 GB or unlimited)
  - Expiry: far future (e.g., 2030-01-01)
  - Inbounds: select both Coruscant + Tatooine
  - Save → copy subscription URL
  - Verify subscription URL returns JSON with 2 outbound configs:
    ```bash
    curl -sk "https://mandalore.severdesign.ru:9443/sub/..." | python3 -m json.tool | grep -c '"outbounds"'
    ```
  - Test: download subscription and verify it contains:
    - Coruscant outbound: VLESS GRPC REALITY, coruscant.severdesign.ru:8443
    - Tatooine outbound: VLESS XHTTP TLS, tatooine.severdesign.ru:2096, SNI dagoba

  **Must NOT do**:
  - Do NOT expose subscription URL in git/docs (it contains a token)
  - Do NOT set unrealistic data limits (user should not hit limits)

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Simple user creation in Marzban panel
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: NO (depends on 15, 16; runs after Task 17)
  - **Parallel Group**: Wave 4 (after Tasks 15, 16)
  - **Blocks**: Task 20 (QA subscription)
  - **Blocked By**: Task 15, Task 16

  **References**:
  - `ACTIVE_SUBSCRIPTIONS.md` — Old subscription URL format for reference
  - `TODO.md` — "only 1 subscription — Rebel Alliance, only 1 user Anakin"

  **Acceptance Criteria**:
  - [ ] Marzban panel shows user "Anakin" with status "active"
  - [ ] Subscription URL returns valid JSON
  - [ ] JSON contains 2 outbounds (Coruscant + Tatooine)
  - [ ] Both outbounds have correct ports and addresses

  **QA Scenarios**:
  ```
  Scenario: Subscription has 2 inbounds
    Tool: Bash (curl from local)
    Preconditions: Subscription URL obtained from panel
    Steps:
      1. curl -sk "SUBSCRIPTION_URL" | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'Outbounds: {len(d.get(\"outbounds\",[]))}') ; [print(f'  {o[\"tag\"]}: {o.get(\"settings\",{}).get(\"vnext\",[{}])[0].get(\"port\",\"?\")}') for o in d.get('outbounds',[])]"
    Expected Result: "Outbounds: 2" with Coruscant port 8443 and Tatooine port 2096
    Evidence: .omo/evidence/task-19-subscription.json (SANITIZED — no tokens)
  ```

  **Evidence to Capture**:
  - [ ] `task-19-subscription.json` (sanitized — remove UUIDs and keys)

  **Commit**: NO

- [x] 20. QA: Panel + subscription endpoint

  **What to do**:
  - Run comprehensive QA on the Marzban panel
  - `curl -sk https://mandalore.severdesign.ru:9443/dashboard/login` → 200
  - `curl -sk https://mandalore.severdesign.ru:9443/api/inbounds` → JSON with Coruscant + Tatooine
  - `curl -sk https://mandalore.severdesign.ru:9443/api/users` → JSON with Anakin
  - `curl -sk "SUBSCRIPTION_URL"` → JSON with 2 outbounds
  - Check subscription outbound details: ports, addresses, protocols match spec

  **Must NOT do**:
  - Do NOT expose full subscription URL in evidence (sanitize tokens)

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: Comprehensive API testing across multiple endpoints
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 5 (with Tasks 21, 22, 23, 24)
  - **Blocks**: Task 25 (docs)
  - **Blocked By**: Task 19

  **References**:
  - Task 19 output — Subscription URL
  - `TODO.md` — Success criteria

  **Acceptance Criteria**:
  - [ ] Panel login page → 200
  - [ ] Inbounds API → 2 inbounds (Coruscant, Tatooine)
  - [ ] Users API → includes Anakin
  - [ ] Subscription → valid JSON with correct port/protocol

  **QA Scenarios**:
  ```
  Scenario: Panel API returns correct state
    Tool: Bash (curl from local)
    Steps:
      1. HTTP_CODE=$(curl -sk -o /dev/null -w "%{http_code}" https://mandalore.severdesign.ru:9443/dashboard/login)
      2. echo "Panel login: $HTTP_CODE" — expect 200
      3. curl -sk https://mandalore.severdesign.ru:9443/api/inbounds | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'Inbounds: {len(d)}')" — expect 2
    Expected Result: 200 + 2 inbounds
    Evidence: .omo/evidence/task-20-panel-qa.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-20-panel-qa.txt`

  **Commit**: NO

- [x] 21. QA: Coruscant inbound (GRPC REALITY)

  **What to do**:
  - TLS handshake check: `openssl s_client -connect coruscant.severdesign.ru:8443 -servername coruscant.severdesign.ru`
  - Verify cert CN = coruscant.severdesign.ru
  - Verify REALITY is responding (should complete TLS handshake successfully)
  - Check from external network (not from Aeza itself)
  - Verify port 8443 responds from internet

  **Must NOT do**:
  - Do NOT test from Aeza localhost (use external machine)

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: TLS/REALITY verification requiring external perspective
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 5 (with Tasks 20, 22, 23, 24)
  - **Blocks**: None
  - **Blocked By**: Task 15

  **References**:
  - `TOPOLOGY.md` — Coruscant port 8443, REALITY config

  **Acceptance Criteria**:
  - [ ] `openssl s_client -connect coruscant.severdesign.ru:8443` returns "Verify return code: 0 (ok)"
  - [ ] Cert subject contains coruscant.severdesign.ru
  - [ ] Port 8443 reachable from external network

  **QA Scenarios**:
  ```
  Scenario: External TLS verification for Coruscant
    Tool: Bash (local machine)
    Steps:
      1. echo | timeout 10 openssl s_client -connect coruscant.severdesign.ru:8443 -servername coruscant.severdesign.ru 2>/dev/null | grep -E "subject=|Verify return code"
    Expected Result: subject=CN=coruscant.severdesign.ru, Verify return code: 0 (ok)
    Failure Indicators: "Connection refused", "no peer certificate", "verify error"
    Evidence: .omo/evidence/task-21-coruscant-tls.txt

  Scenario: GRPC service responds
    Tool: Bash (SSH to Aeza — local check)
    Steps:
      1. ssh skywalker@5.182.86.27 'ss -tlnp | grep ":8443 "'
    Expected Result: Port 8443 listening
    Evidence: .omo/evidence/task-21-coruscant-listen.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-21-coruscant-tls.txt`
  - [ ] `task-21-coruscant-listen.txt`

  **Commit**: NO

- [x] 22. QA: Tatooine inbound (XHTTP TLS)

  **What to do**:
  - TLS handshake: `openssl s_client -connect tatooine.severdesign.ru:2096 -servername dagoba.severdesign.ru`
  - Verify cert includes dagoba.severdesign.ru (or tatooine.severdesign.ru) as SAN
  - Test XHTTP path: `curl -sk https://tatooine.severdesign.ru:2096/api` — should get Xray response (not 404/empty)
  - Check from external network

  **Must NOT do**:
  - Do NOT test from USA localhost

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: XHTTP TLS verification with SNI — requires external perspective
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 5 (with Tasks 20, 21, 23, 24)
  - **Blocks**: None
  - **Blocked By**: Task 16

  **References**:
  - `Atlanta-xhttp-profile-june.json` — XHTTP path `/api`, mode "auto"
  - `TOPOLOGY.md` — Tatooine port 2096, SNI dagoba

  **Acceptance Criteria**:
  - [ ] `openssl s_client -connect tatooine.severdesign.ru:2096 -servername dagoba.severdesign.ru` → Verify return code: 0
  - [ ] `curl -sk https://tatooine.severdesign.ru:2096/api` returns non-empty response
  - [ ] Port 2096 reachable from external network

  **QA Scenarios**:
  ```
  Scenario: External TLS verification for Tatooine with dagoba SNI
    Tool: Bash (local)
    Steps:
      1. echo | timeout 10 openssl s_client -connect tatooine.severdesign.ru:2096 -servername dagoba.severdesign.ru 2>/dev/null | grep -E "subject=|Verify return code"
    Expected Result: subject includes dagoba or tatooine, Verify return code: 0 (ok)
    Evidence: .omo/evidence/task-22-tatooine-tls.txt

  Scenario: XHTTP endpoint responds on /api
    Tool: Bash (local)
    Steps:
      1. curl -sk -o /dev/null -w "%{http_code}" https://tatooine.severdesign.ru:2096/api
    Expected Result: Non-000 response (Xray responds, even if it's a protocol-specific response)
    Evidence: .omo/evidence/task-22-tatooine-xhttp.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-22-tatooine-tls.txt`
  - [ ] `task-22-tatooine-xhttp.txt`

  **Commit**: NO

- [x] 23. QA: Mask redirects (jedha + bespin)

  **What to do**:
  - `curl -sI http://jedha.severdesign.ru` → 301/302 Location: hello.severdesign.ru
  - `curl -sI http://bespin.severdesign.ru` → 301/302 Location: hello.severdesign.ru
  - Test from external network
  - Verify redirect chain completes: follow redirect to hello.severdesign.ru

  **Must NOT do**:
  - Do NOT test from Aeza/USA localhost (use external DNS resolution)

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: Multi-domain redirect chain verification
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 5 (with Tasks 20, 21, 22, 24)
  - **Blocks**: None
  - **Blocked By**: Task 13, Task 14

  **References**:
  - Task 8 output — jedha Nginx config
  - Task 9 output — bespin Nginx config

  **Acceptance Criteria**:
  - [ ] `curl -sI http://jedha.severdesign.ru | grep Location` → hello.severdesign.ru
  - [ ] `curl -sI http://bespin.severdesign.ru | grep Location` → hello.severdesign.ru
  - [ ] Both respond within 5 seconds

  **QA Scenarios**:
  ```
  Scenario: Both masks redirect correctly
    Tool: Bash (local)
    Steps:
      1. JEDHA=$(curl -sI --max-time 5 http://jedha.severdesign.ru | grep -i "^Location:" | tr -d '\r')
      2. BESPIN=$(curl -sI --max-time 5 http://bespin.severdesign.ru | grep -i "^Location:" | tr -d '\r')
      3. echo "Jedha: $JEDHA"
      4. echo "Bespin: $BESPIN"
    Expected Result: Jedha Location: http://hello.severdesign.ru, Bespin Location: http://hello.severdesign.ru
    Failure Indicators: Empty Location, connection timeout, no redirect
    Evidence: .omo/evidence/task-23-mask-redirects.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-23-mask-redirects.txt`

  **Commit**: NO

- [x] 24. QA: AI-service routing via proxy

  **What to do**:
  - Test that AI service domains are routed through proxy:
    - From a test client with the VPN subscription loaded, `curl -x socks5://127.0.0.1:10808 https://api.openai.com` → should go through proxy
    - Or check Marzban panel routing config: verify AI domains → proxy rule
    - Since we can't test client-side easily: verify the routing rules are correctly configured in panel
  - SSH to Sabram Neo and verify routing config: `grep -A5 "openai\|anthropic\|gemini\|adobe\|antigravity" /var/lib/marzban/*.json`
  - Verify direct routing: RU domains → direct rule present

  **Must NOT do**:
  - Do NOT add new routing rules (verify existing ones only)

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: Routing verification requires checking config files and understanding routing logic
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 5 (with Tasks 20, 21, 22, 23)
  - **Blocks**: None
  - **Blocked By**: Task 18

  **References**:
  - Task 18 output — routing configuration
  - `Atlanta-xhttp-profile-june.json` — Expected routing rules

  **Acceptance Criteria**:
  - [ ] AI domains (openai, anthropic, gemini, adobe, antigravity) routed through proxy in Marzban config
  - [ ] RU domains (vk, yandex, mail.ru) routed direct
  - [ ] Apple push domains routed direct
  - [ ] Bittorrent protocol routed direct

  **QA Scenarios**:
  ```
  Scenario: AI services configured for proxy routing
    Tool: Bash (SSH to Sabram Neo)
    Steps:
      1. ssh skywalker@5.42.110.191 'grep -l "openai\|anthropic\|gemini" /var/lib/marzban/*.json /var/lib/marzban/routing* 2>/dev/null'
      2. If found, cat the relevant file and grep for "proxy" near AI domains
    Expected Result: AI domains associated with proxy outbound
    Evidence: .omo/evidence/task-24-ai-routing-check.txt

  Scenario: RU domains configured for direct
    Tool: Bash (SSH)
    Steps:
      1. ssh skywalker@5.42.110.191 'grep -c "vk.com\|yandex" /var/lib/marzban/*.json 2>/dev/null'
    Expected Result: RU domains found with direct outbound tag
    Evidence: .omo/evidence/task-24-ru-routing-check.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-24-ai-routing-check.txt`
  - [ ] `task-24-ru-routing-check.txt`

  **Commit**: NO

- [x] 25. Update all project documentation

  **What to do**:
  - Update `README.md`: Remove warning notice, write as SoT. Document new topology, domains, ports.
  - Update `TOPOLOGY.md`: Replace old topology with new. Coruscant/Aeza, Tatooine/USA, new domains.
  - Update `CURRENT_STATE.md`: Replace with new live state. Remove "broken" language.
  - Update `ACTIVE_SUBSCRIPTIONS.md`: Replace with "1 subscription: Rebel Alliance, 1 user: Anakin". Remove warning.
  - Update `SOURCE_OF_TRUTH.env`: Rewrite with new mapping (PANEL_HOST, DATA_PLANE_HOST_A, DATA_PLANE_HOST_B, all new domains/ports)
  - Update `XRAY_CONFIG.current.json`: Replace with new config summary (Coruscant REALITY, Tatooine XHTTP TLS)
  - Update `OPERATIONS_AND_ROLLBACK.md`: Update health check commands for new domains/ports. Remove warning.
  - Remove `SOURCE_OF_TRUTH.md` (temp copy for Metis — no longer needed)
  - All docs in caveman style, English, short, factual

  **Must NOT do**:
  - Do NOT include secrets, tokens, or keys in docs
  - Do NOT include full subscription URLs
  - Do NOT delete old docs without archiving first (Task 26 handles archive)

  **Recommended Agent Profile**:
  - **Category**: `writing`
    - Reason: Comprehensive documentation update across 7+ files — pure writing task
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: NO (depends on Wave 5 QA passing)
  - **Parallel Group**: Wave 6 (with Tasks 26, 27, 28)
  - **Blocks**: Task 26, Task 28
  - **Blocked By**: Tasks 20-24 (all QA passing)

  **References**:
  - `README.md` — Current (untrusted) state
  - `TOPOLOGY.md` — Old topology
  - `CURRENT_STATE.md` — Old state
  - `ACTIVE_SUBSCRIPTIONS.md` — Old subscriptions
  - `SOURCE_OF_TRUTH.env` — Old mapping
  - `XRAY_CONFIG.current.json` — Old config
  - `OPERATIONS_AND_ROLLBACK.md` — Old operations

  **Acceptance Criteria**:
  - [ ] README.md: documented as SoT, correct topology
  - [ ] TOPOLOGY.md: new servers, domains, ports, transports
  - [ ] CURRENT_STATE.md: reflects working state
  - [ ] ACTIVE_SUBSCRIPTIONS.md: 1 sub, 1 user
  - [ ] SOURCE_OF_TRUTH.env: new mapping complete
  - [ ] XRAY_CONFIG.current.json: new inbounds summary
  - [ ] OPERATIONS_AND_ROLLBACK.md: updated health checks
  - [ ] SOURCE_OF_TRUTH.md removed
  - [ ] All docs in caveman English, no secrets

  **QA Scenarios**:
  ```
  Scenario: No old domain names in active docs
    Tool: Bash (grep)
    Steps:
      1. grep -r "heavymetal\|rage.severdesign\|endor.severdesign\|welcome.severdesign" *.md 2>/dev/null
    Expected Result: No matches (all old names purged from active docs)
    Evidence: .omo/evidence/task-25-no-old-names.txt

  Scenario: No secrets in docs
    Tool: Bash (grep)
    Steps:
      1. grep -rE "(password|secret|token|key|BEGIN.*PRIVATE)" *.md 2>/dev/null | grep -v "TOKEN\|UUID\|cert\|ssl\|\.pem"
    Expected Result: No actual secrets exposed
    Evidence: .omo/evidence/task-25-no-secrets.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-25-no-old-names.txt`
  - [ ] `task-25-no-secrets.txt`

  **Commit**: YES (groups with Task 27)
  - Files: all updated .md files, SOURCE_OF_TRUTH.env, XRAY_CONFIG.current.json

- [x] 26. Archive old files + update .gitignore

  **What to do**:
  - Move old/warning-marked files to Archive/:
    - Keep `README.md` (rewritten), `ACTIVE_SUBSCRIPTIONS.md` (rewritten), `OPERATIONS_AND_ROLLBACK.md` (rewritten)
    - Old versions are already overwritten by Task 25 — add note in Archive about what was changed
    - Move `SOURCE_OF_TRUTH.md` to Archive (temp Metis copy)
    - Move `START_PROMPT.md` from root to Archive (if still exists)
  - Update `.gitignore` to include:
    ```
    Archive/
    backups/
    .env
    *.key
    *.pem
    *.crt
    .omo/evidence/
    .DS_Store
    SOURCE_OF_TRUTH.md
    ```
  - Ensure nothing sensitive is tracked: `git status` should show only docs and configs

  **Must NOT do**:
  - Do NOT delete any files — only move to Archive
  - Do NOT commit Archive/ or backups/

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: File moves and .gitignore update — straightforward
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: NO (depends on Task 25 being complete)
  - **Parallel Group**: Wave 6 (with Tasks 25, 27, 28)
  - **Blocks**: Task 27
  - **Blocked By**: Task 25

  **References**:
  - `.gitignore` from Task 4 — update with additional patterns

  **Acceptance Criteria**:
  - [ ] `SOURCE_OF_TRUTH.md` moved to Archive or deleted
  - [ ] `.gitignore` updated with Archive/, backups/, secrets patterns
  - [ ] `git status` shows only intended files (no secrets, no Archive)

  **QA Scenarios**:
  ```
  Scenario: Git status clean
    Tool: Bash (local)
    Steps:
      1. git status
    Expected Result: Only changed docs/configs, no sensitive files
    Evidence: .omo/evidence/task-26-git-status.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-26-git-status.txt`

  **Commit**: YES (groups with Task 27)

- [x] 27. Git commit + push to "Rise of Republic"

  **What to do**:
  - Stage all changes: `git add -A`
  - Verify staged files: `git diff --cached --stat` — ensure no secrets, no Archive/
  - Commit: `git commit -m "feat: reanimate VPN — new topology with Coruscant + Tatooine"`
  - Push: `git push -u origin "Rise of Republic"`
  - Verify remote: `git log --oneline origin/"Rise of Republic"` shows the commit

  **Must NOT do**:
  - Do NOT push to main/master
  - Do NOT commit secrets
  - Do NOT force push

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Git stage, commit, push — mechanical operation
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - `git-master`: Not needed — operations are straightforward (add, commit, push)

  **Parallelization**:
  - **Can Run In Parallel**: NO (depends on Tasks 25, 26)
  - **Parallel Group**: Wave 6 (with Tasks 25, 26, 28)
  - **Blocks**: None
  - **Blocked By**: Task 26

  **References**:
  - `AGENTS.md` — Remote URL: `git@github.com:Sabramvi/Rebel-Alliance.git`

  **Acceptance Criteria**:
  - [ ] Branch "Rise of Republic" pushed to remote
  - [ ] `git log --oneline -1` shows feat commit
  - [ ] Remote has the commit: `git ls-remote origin "Rise of Republic"` returns a hash

  **QA Scenarios**:
  ```
  Scenario: Commit pushed successfully
    Tool: Bash (local)
    Steps:
      1. git branch --show-current — expect "Rise of Republic"
      2. git log --oneline -1 — expect feat commit message
      3. git push --dry-run origin "Rise of Republic" 2>&1 — expect "Everything up-to-date"
    Expected Result: All 3 checks pass
    Evidence: .omo/evidence/task-27-git-push.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-27-git-push.txt`

  **Commit**: YES
  - Message: `feat: reanimate VPN — Coruscant (REALITY) + Tatooine (XHTTP TLS) inbounds, new domains, routing`
  - Files: All changed docs + configs from Tasks 25, 26

- [x] 28. Create Tolaria knowledge base: Skywalker-VPN-wiki

  **What to do**:
  - Create folder in Tolaria vault: `Skywalker-VPN-wiki/`
  - Create indexed, small notes (caveman style, English):
    - `00-README.md` — Wiki index with table of contents
    - `topology.md` — Current topology (servers, IPs, domains, ports, transports)
    - `operations.md` — Health check commands, SSH commands, restart procedures
    - `routing.md` — Routing rules summary (AI→proxy, RU→direct)
    - `inbounds.md` — Inbound configurations (Coruscant, Tatooine)
    - `domains.md` — DNS mapping table (all domains → IPs)
    - `certs.md` — TLS certificate information (domains, expiry, renewal)
    - `git.md` — Repository info, branch strategy
    - `rollback.md` — Rollback procedures
  - Set proper YAML frontmatter with `type: Note` and wikilinks between notes
  - Use `tolaria_create_note` tool for each note
  - Refresh vault

  **Must NOT do**:
  - Do NOT include secrets, keys, tokens, UUIDs
  - Do NOT create overly large files — keep each note focused
  - Do NOT include outdated information

  **Recommended Agent Profile**:
  - **Category**: `writing`
    - Reason: Creating structured knowledge base with multiple notes — writing-heavy task
  - **Skills**: []
  - **Skills Evaluated but Omitted**:
    - None needed

  **Parallelization**:
  - **Can Run In Parallel**: NO (depends on Task 25 for content)
  - **Parallel Group**: Wave 6 (with Tasks 25, 26, 27)
  - **Blocks**: None
  - **Blocked By**: Task 25

  **References**:
  - `TODO.md` — "make a knowledge base at Tolaria. Make a new folder Skywalker-VPN-wiki"
  - Tolaria vault AGENTS.md — Frontmatter and formatting conventions
  - Task 25 output — Updated documentation (source material)

  **Acceptance Criteria**:
  - [ ] `Skywalker-VPN-wiki/` folder exists in Tolaria vault
  - [ ] At least 8 notes created (README + 7 topics)
  - [ ] All notes have proper YAML frontmatter
  - [ ] Wiki links between notes work
  - [ ] No secrets in any note
  - [ ] Vault refreshed and notes visible

  **QA Scenarios**:
  ```
  Scenario: All wiki notes created
    Tool: Bash (tolaria_search_notes)
    Steps:
      1. Search Tolaria for "Skywalker-VPN-wiki" related notes
      2. Count notes found
    Expected Result: 8+ notes in Skywalker-VPN-wiki/
    Evidence: .omo/evidence/task-28-wiki-notes.txt

  Scenario: No secrets in wiki
    Tool: Bash (grep)
    Steps:
      1. grep -rE "(BEGIN.*PRIVATE|password.*=|secret.*=|token.*=)" "path/to/tolaria/Skywalker-VPN-wiki/" 2>/dev/null
    Expected Result: No matches
    Evidence: .omo/evidence/task-28-no-secrets.txt
  ```

  **Evidence to Capture**:
  - [ ] `task-28-wiki-notes.txt`
  - [ ] `task-28-no-secrets.txt`

  **Commit**: NO (Tolaria is separate from project repo)

---

## Final Verification Wave (MANDATORY — after ALL implementation tasks)

> 4 review agents run in PARALLEL. ALL must APPROVE. Present consolidated results to user and get explicit "okay" before completing.

- [x] F1. **Plan Compliance Audit** — `oracle`
  Read the plan end-to-end. For each "Must Have": verify implementation exists (read file, curl endpoint, run command). For each "Must NOT Have": search codebase for forbidden patterns — reject with file:line if found. Check evidence files exist in .omo/evidence/. Compare deliverables against plan.
  Output: `Must Have [N/N] | Must NOT Have [N/N] | Tasks [N/N] | VERDICT: APPROVE/REJECT`

- [x] F2. **Code Quality Review** — `unspecified-high`
  Review all config files for: hardcoded secrets, syntax errors, unused configs, stale references to old domains/ports. Check nginx configs (`nginx -t`), docker-compose files, Marzban configs. Verify no old domain names (heavymetal, rage, endor, welcome) remain in active configs.
  Output: `Nginx [PASS/FAIL] | Docker [PASS/FAIL] | Configs [N clean/N issues] | VERDICT`

- [x] F3. **Real Manual QA** — `unspecified-high`
  Start from clean state. Execute EVERY QA scenario from EVERY task — follow exact steps, capture evidence. Test cross-task integration (panel + both nodes connected, subscription generates correct configs, inbounds accept connections). Test edge cases: panel restart, node disconnect/reconnect, cert expiry check.
  Output: `Scenarios [N/N pass] | Integration [N/N] | Edge Cases [N tested] | VERDICT`

- [x] F4. **Scope Fidelity Check** — `deep`
  For each task: read "What to do", read actual state (git log/diff, server state). Verify 1:1 — everything in spec was built (no missing), nothing beyond spec was built (no creep). Check "Must NOT do" compliance. Detect cross-task contamination. Flag unaccounted changes.
  Output: `Tasks [N/N compliant] | Contamination [CLEAN/N issues] | Unaccounted [CLEAN/N files] | VERDICT`

---

## Commit Strategy

- **1**: `chore: init git repo with current state` — initial commit on main
- **27**: `feat: reanimate VPN with new topology` — all changes on `Rise of Republic`

---

## Success Criteria

### Verification Commands
```bash
# Panel accessible
curl -sk https://mandalore.severdesign.ru:9443/dashboard/login | head -1
# Expected: HTTP/1.1 200 OK or similar

# Coruscant TLS handshake
openssl s_client -connect coruscant.severdesign.ru:8443 -servername coruscant.severdesign.ru </dev/null 2>/dev/null | grep "Verify return code" 
# Expected: Verify return code: 0 (ok)

# Tatooine TLS handshake
openssl s_client -connect tatooine.severdesign.ru:2096 -servername dagoba.severdesign.ru </dev/null 2>/dev/null | grep "Verify return code"
# Expected: Verify return code: 0 (ok)

# Jedha mask redirect
curl -sI http://jedha.severdesign.ru | grep -i location
# Expected: Location: http://hello.severdesign.ru (or https://)

# Bespin mask redirect
curl -sI http://bespin.severdesign.ru | grep -i location
# Expected: Location: http://hello.severdesign.ru (or https://)
```

### Final Checklist
- [ ] All "Must Have" present
- [ ] All "Must NOT Have" absent
- [ ] Panel accessible on :9443
- [ ] Coruscant inbound working
- [ ] Tatooine inbound working
- [ ] Both masks redirecting
- [ ] Subscription generates 2 inbounds
- [ ] Git pushed on Rise of Republic
- [ ] Tolaria wiki created
