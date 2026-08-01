---
name: Provision a customer account as a Valimail partner
description: Create and configure a customer account, users, and SSO via the Account Management API (partners only).
api: openapi/valimail-account-openapi-original.yml
operations:
  - POST /auth
  - GET /accounts/packages
  - POST /accounts
  - GET /accounts/{slug}
  - POST /accounts/{slug}/users
  - POST /accounts/{slug}/users/{user-slug}/invitation
  - POST /accounts/{slug}/app/sso
  - PUT /accounts/{slug}
generated: '2026-07-21'
method: generated
---

# Provision a customer account as a Valimail partner

Account Management API access is granted to partners only (request via
Valimail Product Support). Specs declare no operationIds — steps use
method + path, verified against `openapi/valimail-account-openapi-original.yml`.

1. **Authenticate.** `POST /auth` with the Account Management API
   `client-id`/`app-id` → bearer token; send `Authorization: Bearer <token>`.
2. **Pick a package.** `GET /accounts/packages` lists the subscription
   packages available to your partner account.
3. **Create the account.** `POST /accounts` creates a customer account with
   your partner account as parent. Confirm with `GET /accounts/{slug}`.
4. **Add users.** `POST /accounts/{slug}/users` (409 means the user already
   exists), then `POST /accounts/{slug}/users/{user-slug}/invitation` to send
   or re-send the invitation email.
5. **Configure SSO (optional).** `POST /accounts/{slug}/app/sso` to create the
   customer's SSO configuration; `PUT`/`DELETE` on the same path manage it.
6. **Adjust subscriptions over time.** `PUT /accounts/{slug}` updates name,
   limits, and package as the customer's needs change; deactivate with the
   v2 delete endpoints (`DELETE /accounts/{slug}/v2`,
   `DELETE /accounts/{slug}/v2/users/{user-slug}`).

Errors use the shared `{request, message, type, request-id, call}` envelope;
`403` signals partner authorization or quota limits, `429` means rate-limited.
