---
name: eclipse-publish-to-open-vsx
description: Publish a VS Code-compatible extension to the Eclipse Open VSX Registry, including namespace creation and token verification. Use when asked to release or update an extension on open-vsx.org. This skill writes — read the safety section before acting.
api: Open VSX Registry API
base_url: https://open-vsx.org
spec: openapi/eclipse-open-vsx-registry-api-openapi.yml
operations:
  - verifyToken
  - createNamespace_1
  - publish_1
  - createAccessToken
  - deleteAccessToken
  - deleteExtension
auth: personal-access-token
---

# Publish to the Eclipse Open VSX Registry

## Safety first — read this before calling anything

This is the only write flow the Eclipse Foundation publishes that an agent is likely to be asked
to drive, and the surface gives you almost no protection:

- **No idempotency.** There is no `Idempotency-Key` header anywhere on the Eclipse API surface.
  If `publish_1` times out, you cannot tell whether it succeeded. Retrying may double-publish.
  Verify state with `getVersions_1` before retrying — never retry blind.
- **No reversal window.** `deleteExtension` exists, but the Foundation publishes no window, no
  soft-delete, and no restore operation. An unpublish is immediate and permanent.
- **Namespace creation is not reversible.** There is no user-facing delete-namespace operation.
- **Get a human to confirm** before the first `publish_1` under a new namespace, and before any
  `deleteExtension`.

See the `idempotency` and `reversibility` blocks in `conventions/eclipse-conventions.yml`.

## Credentials

Publishing needs an Open VSX personal access token, issued at
<https://open-vsx.org/user-settings/tokens> against a free eclipse.org account. It is passed as
the `token` **query parameter**, not as a header — it is not modelled as an OpenAPI
`securityScheme`. The `ovsx` CLI reads it from `OVSX_PAT`. Never put a token in a URL you log.

## Steps

1. **Verify the token before using it.** `verifyToken` — `GET /api/{namespace}/verify-pat?token=…`.
   This is a read, it is safe, and it tells you whether the token is valid for that namespace
   before you attempt any write. Always do this first.

2. **Create the namespace if it does not exist.** `createNamespace_1` —
   `POST /api/-/namespace/create?token=…`. The namespace must match the `publisher` field in the
   extension's `package.json`. Skip if `getNamespace` already returns it.

3. **Publish.** `publish_1` — `POST /api/-/publish?token=…` with the `.vsix` package as the body.
   Expect `201`. A `403` means the token is valid but you are not a member of the namespace —
   membership is not something a scope can substitute for. A `400` means the package failed
   validation.

4. **Confirm.** `getVersions_1` — `GET /api/{namespace}/{extension}/versions`. Do this after every
   publish; it is the only way to establish what actually landed.

## Preferred alternative

For interactive use the Foundation's own CLI is better than raw HTTP, because it packages the
extension for you and keeps the token out of the URL:

```
npx ovsx create-namespace <publisher> --pat $OVSX_PAT
npx ovsx publish --pat $OVSX_PAT
```

See `cli/eclipse-cli.yml`.

## Errors

`400` bad package or request · `401` missing/invalid token · `403` not a namespace member ·
`404` namespace or extension not found · `409` already exists — treat as a soft success for
create-if-absent · `429` rate limited, honour `Retry-After`.
Full catalog: `errors/eclipse-problem-types.yml`.
