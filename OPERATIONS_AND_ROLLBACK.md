# Important notice by USER

This file is NOT A SoT!! Don't trust it! It was written by agent that failed his work and killed system! Just analyse, try understand for context.
Everything written below is a report on the agent’s work that led to a critical system error. Move file after work to archive

## Operations And Rollback

## Health checks

```bash
curl -I https://rage.severdesign.ru/dashboard/
curl -I http://endor.severdesign.ru
openssl s_client -servername endor.severdesign.ru -connect 62.60.244.102:773
```

## Where to run checks

- external TLS check: run from a client machine first
- local node check: SSH to `Aeza`
- panel check: SSH to `Sabram Neo`

## Expected live state

- panel: `200 OK`
- `endor` HTTP: redirect to `welcome.severdesign.ru`
- `endor:773`: valid TLS cert
- `Mos Eisley`: `endor.severdesign.ru:773`
- `Alderaan`: `heavymetal.severdesign.ru:447`

## Rollback idea

1. restore the previous panel DB snapshot if the subscription rows break
2. restore `/var/lib/marzban/xray_config.json` from backup if TLS paths fail
3. restart `marzban-marzban-1`
4. restart the node container on `Aeza` only if the listener still fails

## Do not touch

- `CloudPanel`
- `sabram.ru`
- `Alderaan`
- `443` on `Aeza`
