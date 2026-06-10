# Skywalker-VPN — Reanimation

## Source of Truth

This folder is the canonical handoff for the Skywalker-VPN project (Marzban / Aeza / USA setup).

## Read order

1. `AGENTS.md`
2. `TOPOLOGY.md`
3. `CURRENT_STATE.md`
4. `ACTIVE_SUBSCRIPTIONS.md`
5. `OPERATIONS_AND_ROLLBACK.md`
6. `SOURCE_OF_TRUTH.env`
7. `XRAY_CONFIG.current.json`

## Quick summary

| Component | Domain | Port | Server | Transport |
|---|---|---|---|---|
| Panel | mandalore.severdesign.ru | 9443 | Sabram Neo (5.42.110.191) | HTTPS |
| Coruscant | coruscant.severdesign.ru | 8444 | Aeza (5.182.86.27) | VLESS+GRPC+REALITY |
| Tatooine | tatooine.severdesign.ru | 2096 | USA (184.174.97.95) | VLESS+XHTTP+TLS |
| Jedha mask | jedha.severdesign.ru | 80 | Aeza → hello.severdesign.ru | HTTP redirect |
| Bespin mask | bespin.severdesign.ru | 80 | USA → hello.severdesign.ru | HTTP redirect |
| Subscription | mandalore.severdesign.ru:9443 | — | Sabram Neo | — |

## Constraints

- Do NOT touch CloudPanel (Aeza)
- Do NOT touch sabram.ru (Aeza)
- Do NOT touch port 443 on Aeza (CloudPanel)
- No secrets/keys in git
- Archive/ excluded from git
