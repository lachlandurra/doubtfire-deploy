# Known untested risks — Prototype A (Direct OIDC)

These limitations are explicit so the evidence is not read as production
readiness. The automated tests support application-level identity-continuity
behaviour for the tested flows only.

## Prototype A, Direct OIDC

- **Production redirect-URI / redirect-host lockdown** — `DF_OAUTH_ALLOWED_REDIRECT_HOSTS` is set to `localhost:4200` for development; production allow-listing is not exercised by any test.
- **Browser-level regression across Google and GitHub redirects** — no automated end-to-end (Playwright/Cypress) coverage of the live consent screens; provider redirects are verified manually only.
- **Provider outage handling** — behaviour on Google/GitHub 5xx, timeouts, or expired refresh tokens is untested.
- **Changed-provider identity recovery** — no flow or test for re-linking when a user's provider account changes or a previously used email is reassigned at the provider.
- **Concurrent linking races** — simultaneous link attempts against the same `OauthState` are untested.
- **Production secret management** — client secrets are supplied via environment variables only; no vault/rotation integration is tested.
