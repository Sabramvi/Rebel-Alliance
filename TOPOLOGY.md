# Topology

## Servers

| Name | IP | Role | Services |
|---|---|---|---|
| Sabram Neo | 5.42.110.191 | Control plane | Marzban panel, Nginx, FastPanel |
| Aeza | 5.182.86.27 | Data plane 1 | Marzban-node (Coruscant), CloudPanel, sabram.ru |
| USA | 184.174.97.95 | Data plane 2 | Marzban-node (Tatooine), Nginx — STAGING |

## Domains

| Domain | IP | Purpose |
|---|---|---|
| mandalore.severdesign.ru | 5.42.110.191 | Marzban panel :9443 |
| coruscant.severdesign.ru | 5.182.86.27 | Edge 1 — GRPC REALITY :8444 |
| tatooine.severdesign.ru | 184.174.97.95 | Edge 2 — XHTTP TLS :2096 |
| dagoba.severdesign.ru | 184.174.97.95 | Tatooine SNI |
| jedha.severdesign.ru | 5.182.86.27 | Mask → hello.severdesign.ru |
| bespin.severdesign.ru | 184.174.97.95 | Mask → hello.severdesign.ru |
| hello.severdesign.ru | — | Mask target (legit site) |

## Transports

- **Coruscant**: VLESS + GRPC + REALITY, port 8444, serviceName `cor-grpc`, SNI coruscant.severdesign.ru
- **Tatooine**: VLESS + XHTTP + TLS, port 2096, path `/api`, SNI dagoba.severdesign.ru

## Routing (Xray core config)

- AI services (openai, anthropic, gemini, adobe) — default proxy (through VPN)
- Russian sites (RU/SU/BY domains, geosite:category-ru) — DIRECT
- Apple push — DIRECT
- BitTorrent — DIRECT
- geoip:private — DIRECT

## Check locations

- External checks: from MacBook Air (DNS, TLS, curl)
- Node checks: SSH to Aeza / USA
- Panel checks: SSH to Sabram Neo
