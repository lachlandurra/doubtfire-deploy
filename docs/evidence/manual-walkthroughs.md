# Manual walkthroughs — Prototype A (Direct OIDC)

These checks exercise the live provider redirect flows that the automated suite
cannot reach. They demonstrate development feasibility, not production
reliability. Automated coverage is recorded in
[`prototype-test-traceability.csv`](prototype-test-traceability.csv).

## Prototype A, Continue with Google sign-in

Purpose:
Supports the claim that a previously linked Google identity returns the same
`users.id` rather than creating a new account.

Steps:
1. Seed a student account and sign in normally.
2. From profile settings, link a Google identity (live consent).
3. Sign out, then choose "Continue with Google" on the sign-in page.
4. Complete Google consent and land back on the dashboard.

Expected result:
Session issued for the original `users.id`; no new `User` row.

Actual result:
Pass — resolver maps the Google `sub` to the stored `OauthIdentity` and reuses
the existing token-exchange path.

Evidence location:
Developer walkthrough against seeded accounts; no screenshot retained in-repo.

Why manual:
Live Google redirect and consent screen cannot be reliably automated without a
deployed environment and real provider account.

## Prototype A, Continue with GitHub sign-in

Purpose:
Same as above for the GitHub provider, showing the multi-provider path works.

Steps:
1. Sign in to a seeded student account.
2. Link a GitHub identity from profile settings (live consent).
3. Sign out and choose "Continue with GitHub".
4. Complete GitHub authorisation and return to the dashboard.

Expected result:
Session issued for the original `users.id`; GitHub identity stored in
`oauth_identities`.

Actual result:
Pass.

Evidence location:
Developer walkthrough against seeded accounts.

Why manual:
Live GitHub OAuth redirect is not automatable in the unit-test environment.

## Prototype A, authenticated profile linking

Purpose:
Supports the claim that linking is bound to the signed-in OnTrack user, not to
whatever email the provider returns.

Steps:
1. Sign in via the database auth path.
2. Open profile → identity manager and click "Link Google".
3. Complete consent.

Expected result:
A row in `oauth_identities` owned by the currently signed-in `users.id`.

Actual result:
Pass. The binding rule is also asserted automatically by
`test_linking_flow_attaches_identity_to_authenticated_user_not_email_match`.

Evidence location:
Developer walkthrough plus automated resolver test.

Why manual:
The end-to-end browser link flow is manual; the binding invariant itself is
automated.

## Prototype A, unlink of last sign-in method

Purpose:
Supports the claim that a user cannot remove their only remaining login method.

Steps / status:
Covered by automated tests — `test_unlink_rejected_when_identity_is_last_sign_in_method`
and `test_unlink_allowed_when_user_has_saml_identity`. No separate manual check
was required.
