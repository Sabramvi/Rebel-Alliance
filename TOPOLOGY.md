# Skywalker-VPN Dual-Stream Topology

This document provides a comprehensive and structured overview of the system architecture. The network is split into two entirely independent streams for maximum resilience, isolation, and stealth.

## Stream 1: Aeza Standalone (High Stealth)

A fully isolated standalone system on the Aeza server, implementing a three-layer domain isolation strategy to hide VPN signatures from censors.

**Host:** Aeza (5.182.86.27)

| Layer | Domain | Service | Internal Port | Description |
|---|---|---|---|---|
| **Management** | `coruscant.severdesign.ru` | Marzban Panel | 8000 | Administrative interface. This domain is never visible in VPN data traffic. |
| **Disguise** | `new.severdesign.ru` | Next.js Portfolio | 8080 | Legitimate website. REALITY "steals" the SSL certificate from this domain. |
| **VPN entry** | `jedha.severdesign.ru` | Xray VLESS | 443 (External) | The public endpoint clients connect to. |

### Technical Specifications (Aeza):
- **Transport Stack:** VLESS + XHTTP + REALITY
- **Network Protocol:** `xhttp` (Path: `/hoth`)
- **REALITY Configuration:**
  - **SNI:** `new.severdesign.ru`
  - **Destination:** `127.0.0.1:8443` (Internal Nginx serving the masking site with SSL)
  - **Public/Private Keys:** Freshly generated for Aeza standalone.
- **Client Settings:**
  - **Address:** `jedha.severdesign.ru`
  - **Port:** `443`
  - **Host header:** (Empty or matches SNI)

---

## Stream 2: Neo Cluster (Centrally Managed)

A distributed system managed from a central command server.

**Command & Control:** Sabram Neo (5.42.110.191)
- **Primary Domain:** `mandalore.severdesign.ru`
- **Panel Port:** `51823`

| Data Node | Server IP | VPN Entry Domain | Disguise Domain (SNI) | Transport Type |
|---|---|---|---|---|
| **USA (Tatooine)** | 184.174.97.95 | `tatooine.severdesign.ru` | `severdesign.ru` | VLESS + XHTTP + REALITY |
| **NL (Alderaan)** | 92.51.45.122 | `anakin.design-duo.ru` | `anakin.design-duo.ru` | VLESS + XHTTP + REALITY |

---

## Shared Routing Policies

Both streams use a unified routing logic defined in `CURRENT_ROUTING.json` to ensure seamless access to blocked services while bypassing the VPN for regional traffic.

### Proxy (Strict VPN Routing):
- **AI Ecosystems:** OpenAI (ChatGPT), Anthropic (Claude), Google Gemini, Adobe (Firefly/Creative Cloud), Antigravity.google.
- **Content Platforms:** Midjourney, Suno, Udio, Elevenlabs.

### Direct (Regional & System Traffic):
- **Regional Domains:** `*.ru`, `*.su`, `*.by`, `*.xn--p1ai`.
- **System Services:** Apple Push, BitTorrent, Internal/Private IPs.

---

## User Instructions & Agreements

1. **Isolation:** The Aeza panel and its subscription links are kept strictly separate from the VPN data domains to prevent administrative detection.
2. **Stealth Port:** VPN entry on Aeza uses port `443` to blend with standard HTTPS traffic.
3. **Manual Override:** Technical steps requiring manual execution are provided in the chat for transparency and confirmation.
4. **Standalone Goal:** Aeza remains a fallback system that functions independently of the Sabram Neo control plane.
