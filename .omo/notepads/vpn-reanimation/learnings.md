# Task 4 — git init + initial commit

- Repo remote: git@github.com:Sabramvi/Rebel-Alliance.git
- Branch strategy: main → initial commit → Rise of Republic (all dev work)
- .gitignore excludes: .env, keys/certs, backups/, Archive/, .omo/evidence/, .omo/drafts/, .DS_Store, SOURCE_OF_TRUTH.md
- Initial commit message: "chore: initial state — dead VPN before reanimation"
- No secrets staged — verified via `git diff --cached | grep -E 'password|secret|PRIVATE KEY'`
