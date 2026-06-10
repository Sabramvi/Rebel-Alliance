# Current State

## Topology

- Sabram Neo (5.42.110.191) — control plane. Marzban panel LIVE on mandalore.severdesign.ru:9443.
- Aeza (5.182.86.27) — data plane 1. Marzban-node running, Coruscant inbound on port 8444.
- USA (184.174.97.95) — data plane 2. Ubuntu 24.04, Docker NOT installed yet. Tatooine inbound pending.

## Live services

- Panel: https://mandalore.severdesign.ru:9443/ → 200 OK
- Coruscant inbound: port 8444, REALITY config active
- Jedha mask: Nginx redirect on Aeza, port 80
- CloudPanel (443) and sabram.ru on Aeza — UNTOUCHED, per constraint

## Inbounds (Marzban)

1. **Coruscant** — VLESS GRPC REALITY, port 8444, serviceName cor-grpc, node Aeza
2. Old inbounds still present (773, 447, 448) — pending cleanup after Tatooine setup

## Users

- 1 user: **Anakin** (status: active, UUID generated)
- 1 subscription: **Rebel Alliance** at `/sub/QW5ha2luLDE3ODEwNzExMzgugkEPmvkEm`

## Routing

- RU/SU/BY domains → DIRECT
- Apple push, geoip:private → DIRECT
- BitTorrent → DIRECT
- AI services → default proxy

## Git

- Branch: `rise-of-republic`
- Initial commit: current state
- Remote: git@github.com:Sabramvi/Rebel-Alliance.git

## Blockers

- USA server: skywalker user needs sudo for Docker install
- Tatooine inbound: not configured (USA pending)
- Bespin mask: not configured (USA pending)
- Old inbounds cleanup: pending Tatooine setup
