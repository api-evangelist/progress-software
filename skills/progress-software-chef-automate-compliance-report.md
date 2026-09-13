---
name: Chef Automate — mint a scoped token and pull a compliance report
description: Create an IAM v2 token, scope it with a policy, then list and read Chef InSpec compliance reports out of Chef Automate.
api: openapi/progress-software-chef-automate-openapi-original.json
operations:
  - Tokens_CreateToken
  - Tokens_ListTokens
  - Policies_ListPolicies
  - Policies_AddPolicyMembers
  - ReportingService_ListReports
  - ReportingService_ReadReport
  - Tokens_DeleteToken
---

# Chef Automate — mint a scoped token and pull a compliance report

## Before you start

Chef Automate is **self-hosted**; the base URL is your own Automate installation. The published
contract's `host` is `automate.chef.io`, which is Progress's own instance, not yours.

Authentication is an **API token in the `api-token` header** — not `Authorization`, not a bearer
prefix. The token is minted either with the `chef-automate` CLI
(`chef-automate iam token create <NAME> --admin`) or through the API below.

## 1. Create a token

`POST /apis/iam/v2/tokens` (`Tokens_CreateToken`). A newly created token has **no permissions** until
it is a member of a policy. Do not mint `--admin` tokens for a read-only job.

## 2. Scope it

`GET /apis/iam/v2/policies` (`Policies_ListPolicies`) to find a policy that grants only what you
need, then `POST /apis/iam/v2/policies/{id}/members:add` (`Policies_AddPolicyMembers`) with the
token's member expression. `Policies_ListRoles` and `Policies_ListProjects` show what the
installation actually defines — do not assume the default policy set.

## 3. List reports

`POST /api/v0/compliance/reporting/reports` (`ReportingService_ListReports`). Note the verb: this is
a **POST** that carries a filter body, not a GET. Paginate with `pagination.page` and
`pagination.size`.

## 4. Read one

`POST /api/v0/compliance/reporting/reports/id/{id}` (`ReportingService_ReadReport`) returns the full
InSpec result — profiles, controls, and per-control status.

## 5. Revoke

`DELETE /apis/iam/v2/tokens/{id}` (`Tokens_DeleteToken`) as soon as the job is done.

## Rules that apply to every step

- **Version paths are mixed and the contract does not say which are stable.** One document serves
  `/api/v0/`, `/api/v1/`, `/apis/iam/v2/` and `/api/beta/` concurrently, with no published
  deprecation policy for any of them. `/api/beta/` paths can change; prefer `/apis/iam/v2/` and
  `/api/v0/compliance/` which are what the product documents.
- **Errors are gRPC-shaped.** Chef Automate is a grpc-gateway projection: only 2 of 277 operations
  declare an explicit 4xx. Everything else falls through to the `default` response,
  `grpc.gateway.runtime.Error { code, error, message, details[] }` — and `code` is the **gRPC**
  status code, not the HTTP one. Do not switch on `code` as if it were HTTP.
- **No idempotency key.** A retried `Tokens_CreateToken` mints a second live token. List with
  `Tokens_ListTokens` before retrying.
- **Deleting a policy or project is final** — there is no restore operation and no stated retention
  window.
