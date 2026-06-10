# Important notice by USER

This file is NOT A SoT!! Don't trust it! It was written by agent that failed his work and killed system! Just analyse, try understand for context.
Everything written below is a report on the agent’s work that led to a critical system error. Copy file after work to archive, and update this file

# Skywalker-VPN

This folder is the updated handoff package for the live Marzban / Aeza / Happ setup.

## What to read first

- `AGENTS.md`
- `TOPOLOGY.md`
- `CURRENT_STATE.md`
- `ACTIVE_SUBSCRIPTIONS.md`
- `OPERATIONS_AND_ROLLBACK.md`
- `SOURCE_OF_TRUTH.env`
- `XRAY_CONFIG.current.json`

## Current scope

- `Mos Eisley` uses `xhttp + tls` on `endor.severdesign.ru:773`.
- `Alderaan` stays on `grpc + reality` on `heavymetal.severdesign.ru:447`.
- `endor` HTTP surface redirects to `welcome.severdesign.ru`.
- `rage.severdesign.ru` stays the panel/subscription endpoint.

## Usage

Use this folder as the single source of truth for future edits.
Do not mix it with older handoff folders unless a rollback requires history.
