---
name: Provision an employee into Flash Expense
description: Authenticate a Flash Expense integration user and create or update an employee record, including subsidiary, job position, cost centre, manager and banking data.
api: openapi/flash-app-expense-openapi-original.json
operations:
  - login
  - cadastro-de-usuários
generated: '2026-07-20'
method: generated
source: openapi/flash-app-expense-openapi-original.json
---

# Provision an employee into Flash Expense

Use this skill to sync employees from an HRIS or payroll system into Flash
Expense. Both operations below exist verbatim in Flash's published OpenAPI
definition — do not call anything else, because Flash publishes only these two.

## Before you start

- You need an **integration user** (`usuario integrador`) provisioned by Flash
  for your company, plus your numeric `id_empresa`. There is no self-service
  signup and no API key surface.
- The only base URL Flash declares is the QA host `https://qa.expenseon.com/api`.
  Confirm the production base URL with Flash before running against live data.
- `POST /integration/user` carries employee **banking data** and a `documentId`.
  Treat every payload as personal data under Brazil's LGPD: never log request
  bodies, and do not echo `bankingData` or `documentId` back to a user.

## Step 1 — Authenticate

Call `login` (`POST /login`) with a JSON body:

- `id_empresa` (integer) — your company identifier
- `email` (string) — the integration user's email
- `senha` (string) — the integration user's password
- `recaptcha` (string) — a reCAPTCHA token when the environment requires one

The response schema is not documented; read the session credential from the
response body and reuse it for subsequent calls. If the call returns `400`, the
body is an empty JSON object with no error code — re-check `id_empresa` and the
credentials rather than trying to parse an error message.

## Step 2 — Create or update the employee

Call `cadastro-de-usuários` (`POST /integration/user`) with the employee payload:

- Identity: `idUser`, `name`, `email`, `documentId`, `financeSystemId`
- Placement: `subsidiary` / `refSubsidiary`, `area`, `jobPosition` /
  `refJobPosition`, `refCostCenter`, `refManager`
- Flags: `profile`, `iaActive`
- `bankingData` (object) — the spec declares no properties; get the required
  shape from Flash before sending it

The `ref*` fields are external reference strings into your own system's
identifiers, so send the same values you use upstream to keep the mapping
stable.

## Error and retry rules

- The only declared failure response is `400 application/json` with an empty
  schema. There is no error code registry, no `application/problem+json`, and no
  documented `401`, `404`, `409` or `429`.
- **There is no idempotency key.** Retrying `POST /integration/user` is not
  safe by contract. De-duplicate on your side using `idUser` /
  `financeSystemId` and only retry after confirming the previous call did not
  land.
- No rate limits or rate-limit headers are documented. Batch conservatively and
  back off on any non-200.

## What this API cannot do

There are no list, read, update-by-id or delete operations, no webhooks or
events, and no benefits-side API — the Flash Benefícios reference on the
developer site is still an empty ReadMe template. Do not attempt to construct
endpoints that are not in the spec.
