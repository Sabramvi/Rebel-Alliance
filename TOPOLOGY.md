# Topology

## Hosts

- `Sabram Neo`
  - IP: `5.42.110.191`
  - role: control plane
  - runs: Marzban panel, subscription generation, docs, backups
- `Aeza`
  - IPs: `62.60.244.102` and legacy `85.192.60.50`
  - role: data plane
  - runs: Marzban node, CloudPanel, `sabram.ru`

## Domains

- `rage.severdesign.ru`
  - panel and subscription endpoint
- `heavymetal.severdesign.ru`
  - VPN edge domain
  - resolves to `62.60.244.102`
- `endor.severdesign.ru`
  - Mos Eisley TLS mask domain
  - resolves to `62.60.244.102`
  - HTTP redirects to `welcome.severdesign.ru`
- `welcome.severdesign.ru`
  - legit public site used as mask target

## Transports

- `Mos Eisley`
  - `xhttp + tls`
  - `endor.severdesign.ru:773`
  - `serverName = endor.severdesign.ru`
  - path `/hm`
- `Alderaan`
  - `grpc + reality`
  - `heavymetal.severdesign.ru:447`
  - serviceName `hm-grpc`

## Stable routing

- `Figma` proxy
- `Anthropic / Claude` proxy
- `gorzdrav.spb.ru` direct
- `heavymetal.severdesign.ru` keep as edge

## Check locations

- DNS and TLS checks: use an external machine first, including this MacBook Air.
- Local node checks: SSH to `Aeza` and test `127.0.0.1`.
- Panel checks: SSH to `Sabram Neo`.

