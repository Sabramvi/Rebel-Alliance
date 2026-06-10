# Important notice by USER

This file is NOT A SoT!! Don't trust it! It was written by agent that failed his work and killed system! Just analyse, try understand for context.
Everything written below is a report on the agent’s work that led to a critical system error. Move file after work to archive

## Other Project Agent Bootstrap

Коротко. `caveman ultra`. Для чужого проекта.

## Trigger

Юзер пишет:
`Открой docs/active/other-project-agent-bootstrap.md и приведи этот проект к такой архитектуре.`

## Goal

- Меньше токенов.
- Меньше мусор-docs.
- Больше continuity.
- Агент стартует быстро.
- Агент не читает весь repo без причины.

## Agent Routing

- FULL lane: только анализ, архитектура, риск, миграции, security, финальный review.
- MINI lane: routine work, file moves, docs cleanup, small edits, status checks.
- FULL agents: `Skywalker`, `Boba Fett`, `Mandalorian`.
- MINI agents: `R2D2`, `Fennec Shand`, `Grogu`.
- Если задача мутная / risky -> MINI стоп, FULL план.

## Phase 0: Safety

- Не удалять сразу.
- Сначала inventory.
- Перед важной правкой: git diff backup.
- Секреты не печатать.
- Ключи: только label -> path.

## Phase 1: Project Analysis

- Найти тип проекта.
- Найти entrypoints.
- Найти build/test commands.
- Найти deploy/runtime docs.
- Найти active docs.
- Найти archive/old docs.
- Найти repeated docs.
- Найти contradictory docs.
- Найти secrets refs, не печатать values.
- Сделать `docs/active/project-inventory.md`.

## Phase 2: Docs Inventory

Создать таблицу:

- file path.
- topic.
- status: active / stale / duplicate / archive / unknown.
- action: keep / shorten / merge / archive / ask.
- owner lane: FULL / MINI.

Правило:

- FULL решает truth.
- MINI двигает файлы.
- Unknown не удалять.

## Phase 3: Truth Map

Выбрать canonical docs:

- system map -> `SYSTEM.md`.
- session protocol -> `SESSION.md`.
- current state -> `HANDOFF_CURRENT.md`.
- live plan -> `docs/active/development-plan.md`.
- live snapshot -> `docs/active/session-state.md`.
- routing -> `LLMRoutes.md`.

Если два docs спорят:

- FULL читает оба.
- FULL выбирает truth.
- MINI переносит loser в `docs/archive/`.

## Phase 4: Repo Base

- Если `.git` нет -> `git init`.
- Создать первый baseline commit, если repo пустой.
- Если dirty repo -> показать `git status --short`.
- Не трогать unrelated user changes.

## Phase 5: Minimal Files

Создать / привести:

- `AGENTS.md` - routing + language + lanes.
- `SESSION.md` - start/end protocol.
- `SYSTEM.md` - hosts/users/paths/keys, без секретов.
- `HANDOFF_CURRENT.md` - последний state.
- `docs/active/development-plan.md` - live plan.
- `docs/active/session-state.md` - live snapshot.
- `docs/active/session-bootstrap.md` - короткий start prompt.
- `docs/handoffs/YYYY-MM/` - история handoff.
- `docs/archive/` - старый шум.

## Phase 6: Folder Layout

Привести docs:

- `docs/active/` -> только current truth.
- `docs/active/runbooks/` -> команды и операционка.
- `docs/active/specs/` -> живые specs.
- `docs/active/audits/` -> живые audits.
- `docs/handoffs/YYYY-MM/` -> session history.
- `docs/archive/YYYY-MM/` -> old / stale / duplicate.
- `docs/tmp/` или `tmp/` -> transient scratch.

Правило:

- active docs короткие.
- archive docs не читать на start.
- long logs -> archive.

## Phase 7: LLMRoutes

Создать `LLMRoutes.md`:

- FULL -> analysis / architecture / security / migrations / review.
- MINI -> routine / bounded edits / docs trim / checks.
- Escalation -> unclear scope, prod risk, destructive action, secrets, infra.
- Output -> short, `caveman ultra`.

## Phase 8: Docs Move Plan

Перед переносом сделать план:

- keep list.
- merge list.
- archive list.
- ask list.
- delete candidates.

Потом:

- MINI moves safe archive list.
- FULL resolves ask list.
- Delete only after approval.
- Update links after moves.
- Run `rg` for broken refs.

## Phase 9: Token Policy

- Start reads only:
  - `SESSION.md`
  - `SYSTEM.md`
  - `HANDOFF_CURRENT.md`
  - `docs/active/session-state.md`
  - `docs/active/development-plan.md`
  - `git status --short`
  - `git log -1 --oneline`
- Deep read only on task need.
- Use `rg`, not full repo dump.
- Cache facts in `session-state.md`.

## Phase 10: Session End

- Update `session-state.md`.
- Update `development-plan.md`.
- New handoff -> `docs/handoffs/YYYY-MM/`.
- Copy to `HANDOFF_CURRENT.md`.
- If uncommitted diff -> ask commit now/later.
- If local commit not pushed -> ask push now/later.

## Done When

- New agent can start from 5 files.
- `project-inventory.md` exists.
- Docs classified.
- Docs moved by plan.
- No secret values in docs.
- Routing clear.
- Old docs archived.
- Git state known.
- Token waste lower.
