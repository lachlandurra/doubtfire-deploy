# Prototype C — Magic link (Reading Guide)

This branch of `doubtfire-deploy` carries **Prototype C** of the OnTrack
graduate-access study: passwordless email magic-link access for a personal email
already associated with an OnTrack user. Prototypes A (direct OIDC, branch
`feature/oidc-prototype-a`) and B (Keycloak broker, branch
`feature/prototype-b-evidence`) live on their own branches and are documented
there; this guide covers Prototype C only.

## Branch and submodule pins

| Item | Value |
| --- | --- |
| Parent branch | `feature/magic-link-prototype-c` |
| Evidence tag | `report/prototype-c-final-evidence` |
| Implementation tag | `report/prototype-c-final` |
| `doubtfire-api` submodule | `7af9f8e8eee16282ba1367e29fb9e333222a37ac` (branch `feature/prototype-c-evidence-tests`, off `upstream/10.0.x`) |
| `doubtfire-web` submodule | `6607e312016dea891699ecf42847698a6e27f887` (branch `feature/magic-link-prototype-c`, off `upstream/10.0.x`) |

`git submodule update --init --recursive` lands the tree at the versions evaluated in the report.

## What this prototype is intended to prove

Application-level identity continuity via a verified personal email: a single-use,
time-limited token issued to a pre-associated `personal_email` exchanges for a
normal OnTrack session belonging to the original `users.id`. This is a
supplementary recovery path with weaker identity assurance than OIDC or Keycloak
(mailbox control is the authentication factor). This is development-feasibility
evidence, **not** production reliability.

## Where the evidence is

| Evidence type | Location |
| --- | --- |
| Automated test traceability | [`docs/evidence/prototype-test-traceability.csv`](docs/evidence/prototype-test-traceability.csv) |
| Adoption-cost metrics | [`docs/evidence/prototype-adoption-cost.csv`](docs/evidence/prototype-adoption-cost.csv) |
| Manual walkthroughs | [`docs/evidence/manual-walkthroughs.md`](docs/evidence/manual-walkthroughs.md) |
| Known untested risks | [`docs/evidence/known-untested-risks.md`](docs/evidence/known-untested-risks.md) |

## Running it

1. Check out this branch and run `git submodule update --init --recursive`.
2. In `.devcontainer/devcontainer.env`, set `DF_SMTP_USERNAME`,
   `DF_SMTP_PASSWORD`, and `DF_MAGIC_LINK_FROM_EMAIL` for any SMTP relay
   (defaults assume Gmail via app password; Mailtrap/SendGrid also work — adjust
   `DF_SMTP_ADDRESS` / `DF_SMTP_PORT` / `DF_SMTP_DOMAIN`). Do **not** commit real
   credentials back into the file.
3. **Keep `DF_MAGIC_LINK_AUTO_PROVISION=false`** — this is the fail-closed
   posture: links sent to addresses without an existing OnTrack user are
   rejected. Setting it to `true` lets the Verifier create new accounts and
   breaks the fail-closed claim.
4. Open in VS Code with the Dev Containers extension and run the dev tasks.
5. Run the magic-link evidence tests in the API submodule:
   `bundle exec rails test test/api/magic_link_auth_test.rb test/mailers/magic_link_mailer_test.rb`

## Implementation commits (for orientation)

- API: `feat(magic-link): add passwordless authentication`, `test: add magic link authentication evidence coverage`
- Web: `feat(auth): add magic link sign-in ui`

## Main untested risks (summary)

SMTP deliverability, SPF/DKIM/DMARC, inbox placement, bounce handling,
concurrent rate-limit behaviour, mailbox compromise, and recovery when the
verified personal email is lost. Production requires
`DF_MAGIC_LINK_AUTO_PROVISION=false`.
See [`docs/evidence/known-untested-risks.md`](docs/evidence/known-untested-risks.md).
