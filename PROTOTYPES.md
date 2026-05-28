# Prototype B — Keycloak identity broker (Reading Guide)

This branch of `doubtfire-deploy` carries **Prototype B** of the OnTrack
graduate-access study: a Keycloak identity broker (Google → Keycloak → OnTrack
over OIDC), with OnTrack storing the final linked-login record. Prototypes A
(direct OIDC, branch `feature/oidc-prototype-a`) and C (magic link, branch
`feature/magic-link-prototype-c`) live on their own branches and are documented
there; this guide covers Prototype B only.

## Branch and submodule pins

| Item | Value |
| --- | --- |
| Parent branch | `feature/prototype-b-evidence` |
| Evidence tag | `report/prototype-b-final-evidence` |
| Implementation tag | `report/prototype-b-final` |
| `doubtfire-api` submodule | `b10392204cdbfd6e8fd8d72b2c9e71fe5e53450d` (branch `feature/prototype-b-evidence-tests`, off `upstream/10.0.x`) |
| `doubtfire-web` submodule | `da59b7b80f96c41d6ea7484c66440181e20765b9` (branch `feature/prototype-b`, off `upstream/10.0.x`) |

`git submodule update --init --recursive` lands the tree at the versions evaluated in the report.

## What this prototype is intended to prove

Application-level identity continuity through a broker: a brokered Google
identity resolves through `UserLinkedLogin` back to the original `users.id` and
issues a normal OnTrack session, while provider management lives in Keycloak.
This is development-feasibility and application-level continuity evidence,
**not** production reliability.

## Where the evidence is

| Evidence type | Location |
| --- | --- |
| Automated test traceability | [`docs/evidence/prototype-test-traceability.csv`](docs/evidence/prototype-test-traceability.csv) |
| Adoption-cost metrics | [`docs/evidence/prototype-adoption-cost.csv`](docs/evidence/prototype-adoption-cost.csv) |
| Manual walkthroughs | [`docs/evidence/manual-walkthroughs.md`](docs/evidence/manual-walkthroughs.md) |
| Known untested risks | [`docs/evidence/known-untested-risks.md`](docs/evidence/known-untested-risks.md) |
| Full operator runbook | [`docs/fit4702-runbook.md`](docs/fit4702-runbook.md) |

## Running it

1. Check out this branch and run `git submodule update --init --recursive`.
2. The dev container brings up a local Keycloak at `http://doubtfire-keycloak:8080`
   (host-exposed as `http://localhost:8080`) using realm `doubtfire` and client
   `doubtfire-link`. The realm and client are **auto-imported on first boot**
   from [`development/keycloak/doubtfire-realm.json`](development/keycloak/doubtfire-realm.json)
   via the `--import-realm` flag, and the `DF_KEYCLOAK_*` variables are seeded
   so the OnTrack ↔ Keycloak leg works out of the box.
3. To exercise the **Google → Keycloak** federation leg, register your own
   Google OAuth client in the Keycloak admin UI (one-time, manual). Do **not**
   commit real secrets back into `development/docker-compose.yml`.
4. Open in VS Code with the Dev Containers extension; see
   [`docs/fit4702-runbook.md`](docs/fit4702-runbook.md) for seeded accounts and
   the exact test methods.
5. Run the Keycloak evidence tests in the API submodule:
   `bundle exec rails test test/api/auth_test.rb test/api/users_test.rb`

## Implementation commits (for orientation)

- API: `feat(auth): SAML/Keycloak mapping + Google broker adjustments`, `Modified Keycloak to OIDC Method instead of SAMLAssertions`, `test: add keycloak prototype evidence coverage`, `test(auth): add Keycloak OIDC endpoint test coverage`
- Web: `feat(web): add-keycloak-redirect`, `feat(auth): add "Sign in with Google" button with logo to sign-in page`

## Main untested risks (summary)

Tampered/unsigned state-JWT rejection is not directly asserted; Keycloak upgrade
path, JWKS/key rollover, realm backup, and multi-institution identifier
conflicts are untested; the development realm needs production hardening.
See [`docs/evidence/known-untested-risks.md`](docs/evidence/known-untested-risks.md).
