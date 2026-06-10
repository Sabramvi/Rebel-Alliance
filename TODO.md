# Skywalker-VPN Agent TODO's

This file is the canonical To-Do list. Read it, plan jobs, after my approve - execute

# Changes on domain names/SNI

new canonical names of parts of my project that's a new source of truth. Use it and make changes
EDGE_DOMAIN=heavymetal.severdesign.ru — coruscant.severdesign.ru
EDGE_DOMAIN_SECONDARY (New domain and server) — tatooine.severdesign.ru
PANEL_DOMAIN=rage.severdesign.ru — mandalore.severdesign.ru
MASK_DOMAIN=endor.severdesign.ru — jedha.severdesign.ru
WELCOME_DOMAIN=welcome.severdesign.ru — hello.severdesign.ru
MOS_EISLEY_SNI=endor.severdesign.ru — dagoba.severdesign.ru

Roles of every part — stays the same. Just rename

# Changes (I propose)

New canon: need to change not only names, but ports too
Everywhere in system — every port shall be changed. Please look for stratesig right ports to avoid ad-scanners, DDoSers ana amazon-bots, google bots etc. Especially panel — you should find port, that hasn't interest of such advertising scanners. But when it's a port that should mask node/panel/Marzban/VPN usage — use standart. Propose best variants to make my system stable, clean and not interesting to spam DDoS bots!
Init the GIT repo. Make 1st commit of initial state. Then make new branch named `Rise of Republic`, and all changes make in this branch. push to with. Use VLESS XHTTP TLS for Tatooine, and VLESS GRPC REALITY for Coruscant. Old config provided by old fucktd up agent in XRAY_CONFIG.current.json.
Both configs shall have routing settings like provided in Archive/VPN-vless/Atlanta-xhttp-profile-june.json. AND important! you should add strict settings for all adobe, openai, antropig, gemini, antigravity sites and services to route ONLY over proxy! It's very important!

## New topology

- `Sabram Neo` (`5.42.110.191`) remains control Marzban plane.
- `Aeza` (`5.182.86.27`) remains first data plane (Marzban node). USE for `Endor` inbound.
  — `USA` (`184.174.97.95`) — new second data plane (Marzban node). USE for `Mandalore` inbound.
- `coruscant.severdesign.ru` is the first edge domain and still maps to AEZA `5.182.86.27`.
- `tatooine.severdesign.ru` is the second (new) edge domain and maps to USA `184.174.97.95`.
- `jedha.severdesign.ru` is the Mos Eisley TLS mask domain and also maps to `5.182.86.27`.
- `bespin.severdesign.ru` is the TLS mask domain for USA and also be mapped to `5.182.86.27`

## Documentation after end and Tolaria knowledge base

- after successful finish and deploying/QA — please make a knowledge base at Tolaria. Make a new folder Skywalker-VPN-wiki and make great systematized indexed and easy-to-find-things knowledge base/wiki. The core idea is to provide to agents a pack of small indexed files with main README.md, table of contents etc. Target — pack of well-categorised notes.Don't use old information. It shall be archieved, and .gitignored and agents ignored.

## Targets and success

1. only 1 subscription — `Rebel Alliance`
2. only 2 inbounds — Coruscant and Tatooine
3. Use VLESS XHTTP TLS for Tatooine, and VLESS GRPC REALITY for Coruscant
4. only 1 user Anakin
5. No secret values in docs
6. Docs classified and minimized
