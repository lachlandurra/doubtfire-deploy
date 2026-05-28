# Manual walkthroughs — Prototype C (Magic link)

These checks exercise live SMTP delivery and the real-inbox round trip that the
automated suite cannot reach. They demonstrate development feasibility, not
production reliability. Automated coverage is recorded in
[`prototype-test-traceability.csv`](prototype-test-traceability.csv).

## Prototype C, SMTP delivery end-to-end

Purpose:
Supports the claim that a magic-link email is actually delivered with a
correctly formatted link.

Steps:
1. Set the `DF_SMTP_*` variables for a real relay.
2. Run `rake magic_link:test_delivery EMAIL=<address>` (or trigger issuance from
   the sign-in form).
3. Open the receiving inbox.

Expected result:
A message arrives with the expected subject and a single-use link whose token
sits in the URL fragment query.

Actual result:
Pass against a developer SMTP relay. The link format itself is also asserted by
the mailer tests.

Evidence location:
Developer walkthrough; `lib/tasks/magic_link.rake`. No screenshot retained in-repo.

Why manual:
Live SMTP delivery and inbox receipt cannot be automated in the unit-test
environment (tests use ActionMailer test delivery).

## Prototype C, sign-in via magic link through a real inbox

Purpose:
Supports the claim that consuming a delivered link returns the original
`users.id` to a normal OnTrack session.

Steps:
1. Seed a user with a `personal_email`.
2. Request a magic link for that address from the sign-in form.
3. Open the email and click the link.
4. Land on the dashboard.

Expected result:
Session issued for the original `users.id`; no new account.

Actual result:
Pass. The server-side roundtrip is also asserted by
`test_consume_magic_link_for_existing_personal_email_issues_temporary_token`.

Evidence location:
Developer walkthrough against seeded accounts plus automated roundtrip test.

Why manual:
The email round trip through a real inbox is manual; the token-exchange logic is
automated.

## Prototype C, personal email first-use verification

Purpose:
Documents how `personal_email_verified_at` is set.

Steps / status:
On first successful consumption the `Verifier` stamps
`personal_email_verified_at`. There is **no separate pre-consume verification
flow** — proving mailbox control at consume time is the verification event. This
was confirmed by code inspection of `MagicLinks::Verifier`, not by a dedicated
timestamp-assertion test.

Why manual / noted:
Recorded here for honesty: the report's "verified personal email" wording means
"verified on first use", not a distinct verification step preceding issuance.
