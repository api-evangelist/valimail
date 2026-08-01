---
name: Onboard a domain and manage its DMARC policy
description: Authenticate to the Valimail Configuration API, add a domain, link its senders, and progress its DMARC policy toward enforcement.
api: openapi/valimail-config-openapi-original.yml
operations:
  - POST /auth
  - GET /healthcheck
  - POST /accounts/{slug}/domains
  - GET /accounts/{slug}/domains/{domain}
  - GET /resource/senders
  - POST /accounts/{slug}/domains/{domain}/senders
  - PUT /accounts/{slug}/domains/{domain}
generated: '2026-07-21'
method: generated
---

# Onboard a domain and manage its DMARC policy

The published Valimail specs declare no operationIds — steps reference
method + path, verified against `openapi/valimail-config-openapi-original.yml`.

1. **Authenticate.** `POST /auth` with your `client-id` and `app-id`
   (create keys in the Enforce app under Account Settings → API KEYS; SSO or
   MFA must be enabled). The response contains `token` and `expires-at`. Send
   `Authorization: Bearer <token>` on every later call, and re-authenticate
   before expiry. Optionally verify availability first with `GET /healthcheck`.
2. **Create the domain.** `POST /accounts/{slug}/domains` with
   `{"domain": "example.com"}`. Your account `slug` is visible in the app URL
   between `app/` and `/dmarc`. A `409`-style conflict means it already exists
   — safe to continue.
3. **Review current posture.** `GET /accounts/{slug}/domains/{domain}` returns
   the domain's dmarc-policy, enforcement status, and sending-status.
4. **Link legitimate senders.** Browse the catalog with `GET /resource/senders`,
   then `POST /accounts/{slug}/domains/{domain}/senders` to link each service
   (e.g. SendGrid, Mailgun). `409 Conflict` means the link already exists.
5. **Tighten the policy.** `PUT /accounts/{slug}/domains/{domain}` to update
   `dmarc-policy` (progress none → quarantine → reject) and `sending-status`
   once reporting shows legitimate senders authenticate.

Error handling: errors arrive in a JSON envelope
`{request, message, type, request-id, call}` — include `request-id` in support
tickets. `429` means rate-limited: back off and retry. There is no
idempotency-key contract; rely on the documented `409` conflicts for replay
safety. Test against a sandbox domain/subdomain before touching production
(see `sandbox/valimail-sandbox.yml`).
