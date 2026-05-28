# Prototype A — Direct OIDC via OmniAuth (Reading Guide)

This branch of `doubtfire-deploy` carries **Prototype A** of the OnTrack
graduate-access study: direct OpenID Connect against Google and GitHub,
implemented inside OnTrack via OmniAuth. Prototypes B (Keycloak broker, branch
`feature/prototype-b-evidence`) and C (magic link, branch
`feature/magic-link-prototype-c`) live on their own branches and are documented
there; this guide covers Prototype A only.

## Branch and submodule pins

| Item | Value |
| --- | --- |
| Parent branch | `feature/oidc-prototype-a` |
| Evidence tag | `report/prototype-a-final-evidence` |
| Implementation tag | `report/prototype-a-final` |
| `doubtfire-api` submodule | `8f4f12614bb611603055e0e746a946e14db90502` (off `upstream/10.0.x`) |
| `doubtfire-web` submodule | `78025cd36573ab8a2185abb5f085abfe71c62825` (off `upstream/10.0.x`) |

`git submodule update --init --recursive` lands the tree at the versions evaluated in the report.

## What this prototype is intended to prove

Application-level identity continuity: a current OnTrack user can link a Google
or GitHub identity to their existing `users.id`, and later sign in with that
provider to recover the **same** account — without new identity infrastructure
and without creating duplicate accounts. This is development-feasibility and
application-level continuity evidence, **not** production reliability.

## Where the evidence is

| Evidence type | Location |
| --- | --- |
| Automated test traceability | [`docs/evidence/prototype-test-traceability.csv`](docs/evidence/prototype-test-traceability.csv) |
| Adoption-cost metrics | [`docs/evidence/prototype-adoption-cost.csv`](docs/evidence/prototype-adoption-cost.csv) |
| Manual walkthroughs | [`docs/evidence/manual-walkthroughs.md`](docs/evidence/manual-walkthroughs.md) |
| Known untested risks | [`docs/evidence/known-untested-risks.md`](docs/evidence/known-untested-risks.md) |

## Running it

1. Check out this branch and run `git submodule update --init --recursive`.
2. In `.devcontainer/devcontainer.env`, replace the placeholder
   `DF_OAUTH_GITHUB_CLIENT_ID`, `DF_OAUTH_GITHUB_CLIENT_SECRET`,
   `DF_OAUTH_GOOGLE_CLIENT_ID`, `DF_OAUTH_GOOGLE_CLIENT_SECRET` with your own
   OAuth app credentials. Register the apps at
   <https://github.com/settings/developers> and
   <https://console.cloud.google.com/apis/credentials> with
   `http://localhost:3000/api/auth/oauth/<provider>/callback` as the authorised
   redirect. `DF_OAUTH_PROVIDERS=google,github` controls which buttons appear.
   Do **not** commit real secrets back into this file.
3. Open in VS Code with the Dev Containers extension and run the dev tasks.
4. Run the resolver and auth evidence tests in the API submodule:
   `bundle exec rails test test/services/oauth/identity_resolver_test.rb test/api/auth_test.rb`

## Implementation commits (for orientation)

- API: `feat: add oauth prototype a endpoints`, `feat(oauth): add github provider support`, `feat(oauth): fail closed when no matching OnTrack account exists`, `test: add oauth identity resolver evidence coverage`
- Web: `feat: add google oauth ui`, `feat(oauth): show github provider icon`, `fix(oauth): route link redirects through api origin and surface callback params`

## Main untested risks (summary)

Browser-level provider regression, production redirect-URI lockdown, provider
outage handling, changed-provider recovery, and production secret management.
See [`docs/evidence/known-untested-risks.md`](docs/evidence/known-untested-risks.md).
