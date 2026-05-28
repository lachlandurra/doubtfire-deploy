# Known untested risks — Prototype C (Magic link)

These limitations are explicit so the evidence is not read as production
readiness. The automated tests support token mechanics and application-level
identity continuity for the tested flows only; live email delivery is verified
manually.

## Prototype C, Magic link

- **SMTP deliverability with real providers** — tests use ActionMailer test delivery; behaviour against Gmail/SES/SendGrid is untested.
- **SPF / DKIM / DMARC alignment** — sender-domain authentication is not configured or tested; required before production.
- **Inbox placement** — spam/junk filtering across Gmail/Outlook/institutional MX is untested.
- **Bounce handling** — no handling or test for hard/soft bounces or invalid mailboxes.
- **Concurrent rate-limit behaviour** — the 3-per-120s limit is tested sequentially, not under concurrent load.
- **Mailbox compromise** — magic links inherit email security; a compromised mailbox compromises the OnTrack account. This is an inherent model limitation, not a defect.
- **Recovery when the verified personal email is lost** — no flow or test for re-establishing continuity if the personal mailbox itself is lost.
- **Long-term re-verification of personal_email** — `personal_email_verified_at` is stamped once on first successful consume; no re-verification cadence is defined or tested. There is no separate pre-consume verification flow.
- **Auto-provisioning must stay disabled** — the fail-closed claim (unknown email creates no user) holds only while `DF_MAGIC_LINK_AUTO_PROVISION=false`. With it `true`, the `Verifier` provisions a new account on consume. Production must keep this `false`.
