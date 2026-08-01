---
name: Roll out hosted MTA-STS and TLS-RPT for a domain
description: Publish Valimail-hosted MTA-STS and TLS reporting policies for a domain via the Configuration API, following Valimail's published API guide.
api: openapi/valimail-config-openapi-original.yml
operations:
  - POST /auth
  - POST /accounts/{slug}/domains
  - POST /accounts/{slug}/domains/{domain}/mta_sts_policy
  - POST /accounts/{slug}/domains/{domain}/smtp_tls_policy
  - GET /accounts/{slug}/domains/{domain}/mta_sts_policy
  - GET /accounts/{slug}/domains/{domain}/smtp_tls_policy
  - GET /accounts/{slug}/mta-sts/tls/summary
  - GET /accounts/{slug}/mta-sts/tls/failure-details
generated: '2026-07-21'
method: generated
source: https://support.valimail.com/en/articles/12385962-implement-mta-sts-hosting-tls-reporting-with-valimail-api-guide
---

# Roll out hosted MTA-STS and TLS-RPT for a domain

Mirrors Valimail's published guide (Enforce accounts with API access only).
Specs declare no operationIds — steps use method + path.

1. **Publish the DNS CNAMEs first** (policy hosting depends on them):
   - `_mta-sts.<domain>` CNAME → `<domain>._mta-sts.vali.email`
   - `mta-sts.<domain>` CNAME → `policy.mta-vali.email`
   - `_smtp._tls.<domain>` CNAME → `<domain>._smtptlsrpt.vali.email`
   Allow DNS propagation (~TTL, 60 min) before continuing.
2. **Authenticate.** `POST /auth` with `ClientID`/`AppID` (Configuration API
   credentials) → bearer token + `expires-at`.
3. **Ensure the domain exists.** `POST /accounts/{slug}/domains` with the
   domain name if it is not already in the account.
4. **Create the MTA-STS policy.**
   `POST /accounts/{slug}/domains/{domain}/mta_sts_policy` with
   `policy-mode: "testing"`, your real `mx-hosts`, and `max-age` (e.g. 86400).
   Always start in testing mode; switching to enforce too soon can block mail.
   If you hit "Failed to retrieve SSL certificate expiration date", wait 5–10
   minutes and retry — certificate issuance can lag.
5. **Create the TLS-RPT policy.**
   `POST /accounts/{slug}/domains/{domain}/smtp_tls_policy` with
   `rua-uris: ["mailto:..."]` report destinations.
6. **Validate.** `GET` both policy endpoints, and confirm DNS with dig/curl.
7. **Monitor, then enforce.** Watch `GET /accounts/{slug}/mta-sts/tls/summary`
   and `.../failure-details`; once delivery is clean, `PUT` the MTA-STS policy
   to `policy-mode: "enforce"`.
