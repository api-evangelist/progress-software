---
name: MOVEit Transfer — authenticate and move a file
description: Obtain a MOVEit Transfer bearer token, locate a folder, upload a file into it, and read it back — the core managed-file-transfer loop against a customer's own MOVEit Transfer server.
api: openapi/progress-software-moveit-transfer-openapi-original.json
operations:
  - Auth_GetToken
  - GETapi/v1/folders-1.0
  - POSTapi/v1/folders/{Id}/files?UploadType={UploadType}&PathHash={PathHash}-1.0
  - GETapi/v1/folders/{Id}/files?Page={Page}&PerPage={PerPage}&SortField={SortField}&SortDirection={SortDirection}&Name={Name}&NewOnly={NewOnly}-1.0
  - GETapi/v1/files/{Id}/download-1.0
  - Auth_RevokeToken
---

# MOVEit Transfer — authenticate and move a file

## Before you start

The MOVEit Transfer REST API runs **on the customer's own MOVEit Transfer server**, not on a Progress
host. The base URL is `https://{moveit-transfer-host}/api/v1`. The REST API is an **optionally
licensed feature** — if it is not licensed on that server, every call fails regardless of credentials.

## 1. Get a token

`POST /api/v1/token` (`Auth_GetToken`) with an OAuth2 password-grant form body. The response carries
`access_token`. Send it on every subsequent call as `Authorization: Bearer {token}`.

Three auth outcomes are first-class in the contract and you must branch on them, not just on the
status code:

- `400` with `RequestTokenPasswordChangeRequiredError` — the account must change its password first.
- `400` with `RequestTokenSecurityNoticeAcceptanceRequiredError` — a security notice must be accepted.
- `401` with `RequestTokenMfaError`, or `412` requiring MFA setup — call `Auth_SendOtp`
  (`POST /api/v1/token/otp`) and repeat the token request with the OTP.

## 2. Find the folder

`GET /api/v1/folders` lists the folders visible to the token's user. Each result carries an `id` and
a `pathHash`. Many folder operations accept `PathHash` as a disambiguator — pass it through
unchanged; do not construct one.

Paginate with `Page` and `PerPage`, sort with `SortField` / `SortDirection`. The response envelope is
`{ paging, sorting, items }`.

## 3. Upload

`POST /api/v1/folders/{Id}/files` with `UploadType` and, where the folder needs it, `PathHash`. The
body is a multipart file upload. The response carries the new file `id`.

## 4. Read back

- `GET /api/v1/folders/{Id}/files` to list, filtering with `Name` and `NewOnly`.
- `GET /api/v1/files/{Id}/download` to retrieve content.

## 5. Clean up

`POST /api/v1/token/revoke` (`Auth_RevokeToken`) when the session is done.

## Rules that apply to every step

- **There is no idempotency key.** The MOVEit Transfer contract has no `Idempotency-Key` header and
  no documented replay window. A retried upload creates a second file. Before retrying any write,
  re-read the folder listing and check whether the first attempt landed.
- **Deletion is not reversible through this API.** `DELETE /api/v1/files/{Id}` has no restore
  counterpart in the contract, and `DELETE /api/v1/mailboxes/trash` *empties* the trash rather than
  restoring from it. Treat every delete as final.
- **Errors** come back as `ErrorModel { title, detail, errorCode }` as `application/json` — it is
  Problem-Details-shaped but is **not** RFC 9457, so do not look for a `type` URI. A `422` returns
  `UnprocessableEntityErrorModel` with `errors[]` of `FieldErrorModel { field, rejected, message }`;
  read `field` to know which input to fix. `403 Forbidden` and `500 Server Error` are declared on
  nearly every operation (91 and 92 of 113 respectively).
- **No rate-limit signal.** No `RateLimit-*` or `Retry-After` header exists and no 429 is declared.
  Back off on your own schedule.
