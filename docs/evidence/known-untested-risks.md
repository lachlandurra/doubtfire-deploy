# Known untested risks — Prototype B (Keycloak broker)

These limitations are explicit so the evidence is not read as production
readiness. The automated tests support application-level identity-continuity
behaviour for the tested flows only; the live Google → Keycloak browser redirect
is verified manually with stubbed HTTP in tests.

## Prototype B, Keycloak broker

- **Tampered or unsigned state-JWT rejection** — the signed OAuth state is generated and verified on every callback (via `AuthenticationHelpers.generate_oauth_state` / `verify_oauth_state`), but **no dedicated test asserts that a tampered or unsigned state is rejected**. This rejection branch is currently code-review-only.
- **Keycloak upgrade path** — Keycloak is pinned to `24.0`; behaviour across version upgrades is untested.
- **JWKS / signing-key rollover** — ID-token verification uses the JWKS endpoint, but key rotation is not exercised.
- **Realm backup and restore** — only first-boot import from the committed realm JSON is verified; backup/restore of a live `keycloak_data` volume is untested.
- **Multi-institution identifier conflicts** — whether the same provider identifier can collide across realms/institutions is untested.
- **Production hardening of the development realm** — the imported realm uses development defaults (admin/admin, fixed client secret, localhost redirect URIs). Production needs secret rotation, redirect-URI lockdown, and removal of default admin credentials.
- **Browser-level Google → Keycloak regression** — no automated end-to-end coverage of the live redirect chain.
