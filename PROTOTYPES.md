# OnTrack OIDC Prototypes — Reading Guide

This fork of `doubtfire-deploy` carries three federated-identity prototypes that accompany the IEEE report. Each prototype lives on its own branch in this repo **and** in the two submodules. A fresh clone of any prototype branch with `--recurse-submodules` gives a buildable tree.

## Branches

| Prototype | Mechanism | Parent branch | Submodule branches |
| --- | --- | --- | --- |
| **A** | Direct OIDC against Google + GitHub | `feature/oidc-prototype-a` | `doubtfire-api: feature/oidc-prototype-a`, `doubtfire-web: feature/oidc-prototype-a` |
| **B** | Keycloak broker (Google → Keycloak → OnTrack via OIDC) | `feature/prototype-b-evidence` | `doubtfire-api: feature/prototype-b-evidence-tests`, `doubtfire-web: feature/prototype-b` |
| **C** | Email magic link (passwordless) | `feature/oidc-prototype-c` | `doubtfire-api: feature/prototype-c-evidence-tests`, `doubtfire-web: feature/oidc-prototype-c` |

Each parent branch records the corresponding submodule commits so `git submodule update --init --recursive` lands the tree at the version evaluated in the report.

## Running a prototype

1. Check out the prototype branch on this repo and run `git submodule update --init --recursive`.
2. Copy `.devcontainer/devcontainer.env` and replace the placeholder values for the prototype you are running:
   - **Prototype A** requires real values for `DF_OAUTH_GITHUB_CLIENT_ID`, `DF_OAUTH_GITHUB_CLIENT_SECRET`, `DF_OAUTH_GOOGLE_CLIENT_ID`, and `DF_OAUTH_GOOGLE_CLIENT_SECRET`. Create OAuth apps at <https://github.com/settings/developers> and <https://console.cloud.google.com/apis/credentials> and register `http://localhost:3000/api/auth/oauth/<provider>/callback` as the authorised redirect.
   - **Prototype B** requires a running Keycloak realm with Google federation configured. The Keycloak issuer URL and client credentials go in the `DF_KEYCLOAK_*` variables. See the report § IV-C for the realm export used during evaluation.
   - **Prototype C** requires SMTP credentials (`SMTP_*`) so the magic link mailer can deliver. Any reliable inbox provider works; we used Mailtrap during evaluation.
3. Open the project in VS Code with the Dev Containers extension and run the standard dev tasks.
4. Tests for each prototype live in the submodule's `test/` directory and run with `bundle exec rails test test/services/oauth/identity_resolver_test.rb` (Prototype A) or the equivalent paths for B and C.

## Evidence locations

| Item | Prototype A | Prototype B | Prototype C |
| --- | --- | --- | --- |
| Implementation commits (API) | `feat(oauth): add github provider support`, `feat: add oauth prototype a endpoints`, `feat(oauth): fail closed when no matching OnTrack account exists` | `feat(auth): SAML/Keycloak mapping + Google broker adjustments`, `Modified Keycloak to OIDC Method instead of SAMLAssertions` | `feat(magic-link): add passwordless authentication` |
| Test commit (API) | `test: add oauth identity resolver evidence coverage` | `test: add keycloak prototype evidence coverage` | `test: add magic link authentication evidence coverage` |
| Implementation commits (web) | `feat: add google oauth ui`, `feat(oauth): show github provider icon`, `fix(oauth): route link redirects through api origin and surface callback params` | `feat(web): add-keycloak-redirect` | `feat(auth): add magic link sign-in ui` |

## Backup tags

Every pre-cleanup branch tip is preserved as `backup/pre-cleanup/<branch>` in each of the three repos. The current parent + submodule pointers are the versions referenced from the IEEE report.
