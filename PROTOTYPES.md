# OnTrack OIDC Prototypes — Reading Guide

This fork of `doubtfire-deploy` carries three federated-identity prototypes that accompany the IEEE report. Each prototype lives on its own branch in this repo **and** in the two submodules. A fresh clone of any prototype branch with `--recurse-submodules` gives a buildable tree.

## Branches

| Prototype | Mechanism | Parent branch | Submodule branches |
| --- | --- | --- | --- |
| **A** | Direct OIDC against Google + GitHub | `feature/oidc-prototype-a` | `doubtfire-api: feature/oidc-prototype-a`, `doubtfire-web: feature/oidc-prototype-a` |
| **B** | Keycloak broker (Google → Keycloak → OnTrack via OIDC) | `feature/prototype-b-evidence` | `doubtfire-api: feature/prototype-b-evidence-tests`, `doubtfire-web: feature/prototype-b` |
| **C** | Email magic link (passwordless) | `feature/magic-link-prototype-c` | `doubtfire-api: feature/prototype-c-evidence-tests`, `doubtfire-web: feature/magic-link-prototype-c` |

Each parent branch records the corresponding submodule commits so `git submodule update --init --recursive` lands the tree at the version evaluated in the report.

## Running a prototype

1. Check out the prototype branch on this repo and run `git submodule update --init --recursive`.
2. Edit `.devcontainer/devcontainer.env` for the prototype you are running. The file already ships with sensible defaults for everything except the secrets a marker has to provide:
   - **Prototype A** — direct OIDC. Replace the placeholder values for `DF_OAUTH_GITHUB_CLIENT_ID`, `DF_OAUTH_GITHUB_CLIENT_SECRET`, `DF_OAUTH_GOOGLE_CLIENT_ID`, and `DF_OAUTH_GOOGLE_CLIENT_SECRET`. Register the OAuth apps at <https://github.com/settings/developers> and <https://console.cloud.google.com/apis/credentials> with `http://localhost:3000/api/auth/oauth/<provider>/callback` as the authorised redirect. `DF_OAUTH_PROVIDERS=google,github` controls which providers appear on the sign-in page; trim that list if you only want one. The scopes and redirect URIs in the file are correct as-is.
   - **Prototype B** — Keycloak broker. The dev container brings up a local Keycloak at `http://doubtfire-keycloak:8080` (host-exposed as `http://localhost:8080`) using realm `doubtfire` and client `doubtfire-link`; the realm + client are auto-imported on first boot from `development/keycloak/doubtfire-realm.json`, and the `DF_KEYCLOAK_*` variables are already seeded so the OnTrack ↔ Keycloak leg works out of the box. To exercise the **Google → Keycloak** federation leg, replace `GOOGLE_CLIENT_ID` with a real Google OAuth app ID and register it in the Keycloak admin UI (one-time, manual). See [`docs/fit4702-runbook.md`](docs/fit4702-runbook.md) for the full Prototype B walkthrough, including seeded test accounts and the exact test methods to invoke.
   - **Prototype C** — magic link. Replace `DF_SMTP_USERNAME`, `DF_SMTP_PASSWORD`, and `DF_MAGIC_LINK_FROM_EMAIL` with credentials for any SMTP relay (the defaults assume Gmail via app password; Mailtrap, SendGrid, etc. all work — just adjust `DF_SMTP_ADDRESS` / `DF_SMTP_PORT` / `DF_SMTP_DOMAIN`). The link TTL, resend window, max attempts and callback URL are pre-configured in `DF_MAGIC_LINK_*`. **Note**: `DF_MAGIC_LINK_AUTO_PROVISION=false` is the fail-closed posture — magic links sent to addresses without an existing OnTrack user are rejected. This is the Prototype C counterpart to Prototype A's resolver change.
3. Open the project in VS Code with the Dev Containers extension and run the standard dev tasks.
4. Tests for each prototype live in the submodule's `test/` directory:
   - Prototype A: `bundle exec rails test test/services/oauth/identity_resolver_test.rb`
   - Prototype B: see the keycloak coverage added in commits `934e6d16` (`test: add keycloak prototype evidence coverage`) and `b1039220` (`test(auth): add Keycloak OIDC endpoint test coverage`)
   - Prototype C: see the magic-link coverage added in commit `7af9f8e8` (`test: add magic link authentication evidence coverage`)

## Evidence locations

| Item | Prototype A | Prototype B | Prototype C |
| --- | --- | --- | --- |
| Implementation commits (API) | `feat(oauth): add github provider support`, `feat: add oauth prototype a endpoints`, `feat(oauth): fail closed when no matching OnTrack account exists` | `feat(auth): SAML/Keycloak mapping + Google broker adjustments`, `Modified Keycloak to OIDC Method instead of SAMLAssertions` | `feat(magic-link): add passwordless authentication` |
| Test commit (API) | `test: add oauth identity resolver evidence coverage` | `test: add keycloak prototype evidence coverage`, `test(auth): add Keycloak OIDC endpoint test coverage` | `test: add magic link authentication evidence coverage` |
| Implementation commits (web) | `feat: add google oauth ui`, `feat(oauth): show github provider icon`, `fix(oauth): route link redirects through api origin and surface callback params` | `feat(web): add-keycloak-redirect`, `feat(auth): add "Sign in with Google" button with logo to sign-in page` | `feat(auth): add magic link sign-in ui` |

## Backup tags

Every pre-cleanup branch tip is preserved as `backup/pre-cleanup/<branch>` in each of the three repos. The current parent + submodule pointers are the versions referenced from the IEEE report.
