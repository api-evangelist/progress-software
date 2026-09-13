---
name: MOVEit Transfer — send an ad-hoc secure package
description: Compose, attach files to, and send an Ad Hoc Transfer package from a MOVEit Transfer mailbox, then track recipients and the notification that was sent.
api: openapi/progress-software-moveit-transfer-openapi-original.json
operations:
  - Auth_GetToken
  - GETapi/v1/packages/requirements-1.0
  - POSTapi/v1/packages/attachments-1.0
  - POSTapi/v1/packages-1.0
  - GETapi/v1/packages/{Id}?Action={Action}&MailboxId={MailboxId}-1.0
  - GETapi/v1/packages/{Id}/recipients?MailboxId={MailboxId}-1.0
  - GETapi/v1/packages/{Id}/notification?IncludeImages={IncludeImages}&MailboxId={MailboxId}-1.0
---

# MOVEit Transfer — send an ad-hoc secure package

Ad Hoc Transfer is MOVEit's person-to-person secure send. A *package* is a message plus attachments
addressed to recipients, delivered through a MOVEit mailbox.

## 1. Authenticate

`Auth_GetToken` — see the authenticate-and-transfer skill for the MFA / password-change /
security-notice branches. Send `Authorization: Bearer {token}` on everything below.

## 2. Ask the server what a package requires

`GET /api/v1/packages/requirements`. This returns the server's configured constraints — the
administrator, not you, decides what is mandatory. **Call this first and obey it**; the Ad Hoc
settings surface (`/api/v1/settings/adhoctransfer/...`) differs per installation.

## 3. Upload attachments

`POST /api/v1/packages/attachments` for each file, before the package exists. Keep the returned
attachment ids. `DELETE /api/v1/packages/attachments/{Id}` removes one that has not been sent yet.

## 4. Send

`POST /api/v1/packages` with recipients, subject, body and the attachment ids from step 3.

## 5. Track

- `GET /api/v1/packages/{Id}` — pass `Action` and `MailboxId` to scope the view.
- `GET /api/v1/packages/{Id}/recipients` — who it went to.
- `GET /api/v1/packages/{Id}/notification` — the notification message that was generated.
- `GET /api/v1/mailboxes/{Id}/packages` — everything in one mailbox.

## Rules

- **Sending is not reversible.** `DELETE /api/v1/packages/{Id}` removes the package record; it does
  not unsend a notification that already left the server, and the contract states no window in which
  it would. Confirm the recipient list before the POST in step 4, not after.
- **No idempotency key.** A retried `POST /api/v1/packages` sends a second package. On a timeout,
  list the mailbox with `GET /api/v1/mailboxes/{Id}/packages` and check before resending.
- Validation failures return `422` with `errors[]` of `FieldErrorModel { field, rejected, message }`.
