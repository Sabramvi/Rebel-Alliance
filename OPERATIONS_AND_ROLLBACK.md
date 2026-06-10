# Operations And Rollback

## Health checks

```bash
# Panel
curl -sk https://mandalore.severdesign.ru:9443/
# Expected: 200 (SPA shell)

# Coruscant REALITY
echo | openssl s_client -connect coruscant.severdesign.ru:8444 -servername coruscant.severdesign.ru 2>/dev/null | grep "Verify return code"
# Expected: 0 (ok)

# Jedha mask
curl -sI http://jedha.severdesign.ru | grep Location
# Expected: hello.severdesign.ru

# Bespin mask
curl -sI http://bespin.severdesign.ru | grep Location
# Expected: hello.severdesign.ru
```

## Where to run checks

- External TLS/DNS checks: from a client machine (MacBook)
- Node checks: SSH to Aeza (`ssh -J sabram-crm -i ~/.ssh/aeza-skywalker skywalker@5.182.86.27`)
- USA checks: SSH via `ssh -i ~/.ssh/skywalker_deploy skywalker@184.174.97.95`
- Panel checks: SSH to Sabram Neo (`ssh sabram-crm`)

## Expected live state

- Panel: 200 OK on mandalore.severdesign.ru:9443
- Coruscant: REALITY handshake on coruscant.severdesign.ru:8444
- Jedha: HTTP redirect to hello.severdesign.ru
- Bespin: HTTP redirect to hello.severdesign.ru (PENDING — USA setup)
- Tatooine: XHTTP TLS on tatooine.severdesign.ru:2096 (PENDING)

## Rollback steps

1. Restore `/opt/marzban/.env` from backup (`backups/sabram-neo/.env.bak`)
2. Restore `/var/lib/marzban/db.sqlite3` from backup dump
3. Restore `/opt/marzban/docker-compose.yml` from backup
4. `cd /opt/marzban && sudo docker compose down && sudo docker compose up -d`
5. If Aeza node fails: restore docker-compose.yml from `backups/aeza/marzban-node/`

## Do NOT touch

- CloudPanel (Aeza)
- sabram.ru (Aeza)
- Port 443 on Aeza
