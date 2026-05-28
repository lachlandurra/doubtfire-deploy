# Manual walkthroughs — Prototype B (Keycloak broker)

These checks exercise the live browser redirect and infrastructure boot that the
automated suite cannot reach. They demonstrate development feasibility, not
production reliability. Automated coverage is recorded in
[`prototype-test-traceability.csv`](prototype-test-traceability.csv); the full
operator walkthrough is in [`fit4702-runbook.md`](../fit4702-runbook.md).

## Prototype B, Google through Keycloak sign-in

Purpose:
Supports the claim that a brokered Google identity returns the original
`users.id` through `UserLinkedLogin`.

Steps:
1. `docker compose up` so Keycloak boots and imports realm `doubtfire`.
2. On the OnTrack sign-in page choose "Sign in with Google".
3. OnTrack redirects to Keycloak, which redirects to Google consent.
4. Complete consent and land back on the OnTrack dashboard.

Expected result:
Session issued for the pre-linked `users.id`; no new account.

Actual result:
Pass — the callback resolves the brokered identity via `UserLinkedLogin`. The
resolution and one-time-token issuance are also asserted by
`test_keycloak_google_signin_callback_uses_linked_login_and_issues_one_time_token`.

Evidence location:
`docs/fit4702-runbook.md`; developer walkthrough against seeded accounts.

Why manual:
The live Google → Keycloak browser redirect cannot be reliably automated without
a deployed environment; tests stub the Keycloak/Google HTTP responses.

## Prototype B, authenticated Link Google flow

Purpose:
Supports the claim that linking is bound to the signed-in user and stored in
OnTrack.

Steps:
1. Sign in via the database auth path.
2. Start the "Link Google" flow from the account area.
3. Pass through Keycloak/Google and return to OnTrack.

Expected result:
A `user_linked_logins` row owned by the current `users.id`.

Actual result:
Pass. The record-write invariant is also asserted by
`test_keycloak_google_link_callback_records_linked_login`.

Evidence location:
`docs/fit4702-runbook.md`; developer walkthrough.

Why manual:
Live broker redirect; only the server-side callback is automated (stubbed HTTP).

## Prototype B, clean-volume Keycloak realm import

Purpose:
Supports the claim that the realm configuration is reproducible, not hand-built.

Steps:
1. `docker compose down -v` to drop the `keycloak_data` volume.
2. `docker compose up keycloak`.
3. Inspect the Keycloak admin UI for realm `doubtfire` and client `doubtfire-link`.

Expected result:
Realm and client present on first boot, imported from
`development/keycloak/doubtfire-realm.json`.

Actual result:
Pass — `--import-realm` loads the committed realm JSON.

Evidence location:
`development/docker-compose.yml` (`--import-realm`);
`development/keycloak/doubtfire-realm.json`.

Why manual:
Container boot and realm import is infrastructure behaviour, not a unit test.

## Prototype B, linked-login list / unlink in the UI

Purpose:
Supports the claim that identity-management endpoints enforce ownership in the
running app.

Steps:
1. Sign in and open the linked-logins view.
2. List linked logins, then unlink one.

Expected result:
Only the signed-in user's logins are shown and removable.

Actual result:
Pass. Ownership is also asserted by `test_get_linked_logins_rejects_other_student`
and `test_delete_linked_login_rejects_other_user`.

Evidence location:
Developer walkthrough; automated authorisation tests in `users_test.rb`.

Why manual:
The UI integration is manual; the authorisation invariants are automated.
