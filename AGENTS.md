# Skywalker-VPN Agent Rules

This folder is the canonical handoff for the infra project.

Now let's fix my currently dead VPN. Read the documentation, I've marked my wishes and proposals. Especially watch TODO.md. Anakin wait plan

## Rules

- Write all documentation in English.
- Use caveman style for documentation in this folder.
- When chatting with the user, speak Russian in caveman ultra.
- Keep docs short, factual, and current.
- do not touch `CloudPanel`, `sabram.ru`

## Read order

1. `README.md`
2. `TOPOLOGY.md`
3. `CURRENT_STATE.md`
4. `ACTIVE_SUBSCRIPTIONS.md`
5. `OPERATIONS_AND_ROLLBACK.md`
6. `TODO.md`
7. `SOURCE_OF_TRUTH.env`
8. `XRAY_CONFIG.current.json`

## Topology rules (old)

- `Sabram Neo` is the control plane.
- `Aeza` is the data plane, and first Marzbannode.
- `rage.severdesign.ru` is the panel/subscription endpoint.
- `heavymetal.severdesign.ru` is the edge domain.
- `endor.severdesign.ru` is the Mos Eisley TLS mask domain.
- `welcome.severdesign.ru` is the legit mask target.
- git@github.com:Sabramvi/Rebel-Alliance.git

## Topology rules (to do)

While need to watch topology scheme — read TODO.md, ## New topology

## Rules

- Do not touch `CloudPanel`.
- Do not touch `sabram.ru`.
- no passwords/secrets etc in git! No Archive in git! No Archive folder in everytime-reading. read only that and then, thai I say. Tolaria vault — the same rule

## At finish

- handoff, updated file/folder structure and docs. Outdated information/files stored in Archive folder (both repo/tolaria)
- properly and effectively filled, indexed knowledge base at Tolaria
- git commited and pushed
- the whole `Rebel Alliance` system works properly (approved by user)
- current state updated, docs updated
- merge only after my approval (my name in this project is Anakin)
