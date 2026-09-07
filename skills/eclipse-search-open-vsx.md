---
name: eclipse-search-open-vsx
description: Search, resolve and download VS Code-compatible extensions from the Eclipse Open VSX Registry. Use when asked to find an extension, check its versions or target platforms, get its download URL, or compare it with the Microsoft marketplace.
api: Open VSX Registry API
base_url: https://open-vsx.org
spec: openapi/eclipse-open-vsx-registry-api-openapi.yml
operations:
  - search
  - getQueryV2
  - getNamespace
  - getNamespaceDetails
  - getExtension
  - getExtension_1
  - getExtension_2
  - getExtension_3
  - getVersions
  - getVersions_1
  - getVersionReferences_1
  - getReviews
  - getChanges
  - getRegistryVersion
auth: none
---

# Search the Eclipse Open VSX Registry

Open VSX is the Eclipse Foundation's vendor-neutral registry for VS Code-compatible extensions.
All read operations below are anonymous.

## Steps

1. **Search.** `search` — `GET /api/-/search`. Paginates by `offset` and `size` (NOT `page`/
   `pagesize` — Open VSX does not share the pagination convention used by the Eclipse Foundation
   IT APIs). The response envelope carries `offset` and `totalSize`. `getQueryV2` —
   `GET /api/v2/-/query` — is the structured query form.

   Do not use `postQuery` (`POST /api/-/query`); it is flagged `deprecated: true` in the
   specification, superseded by `getQueryV2`.

2. **Resolve the extension.** An extension is keyed by the composite
   `{namespace}/{extension}`, optionally narrowed by `{targetPlatform}` and `{version}`:
   - `getExtension` — `GET /api/{namespace}/{extension}` (latest)
   - `getExtension_1` — `GET /api/{namespace}/{extension}/{version}`
   - `getExtension_2` / `getExtension_3` — the target-platform-qualified forms

3. **List versions.** `getVersions_1` — `GET /api/{namespace}/{extension}/versions`, or
   `getVersions` for a specific target platform. `getVersionReferences_1` returns the version
   reference list without full metadata, which is cheaper when you only need the version numbers.

4. **Read the namespace.** `getNamespace` — `GET /api/{namespace}` — and `getNamespaceDetails` —
   `GET /api/{namespace}/details` — for ownership and the published extension list.

5. **Track change.** `getChanges` — `GET /api/-/version-changes` — is the registry's
   version-change feed, and the only change stream published anywhere on the Eclipse API surface.
   `getRegistryVersion` — `GET /api/version` — is a cheap probe for the running server version.

## The VS Code gallery protocol

Open VSX also implements the Microsoft VS Code Extension Gallery wire protocol under `/vscode/*`
(`extensionQuery`, `getLatest`, `download`, `getAsset`, `browse`, `getItemUrl`). If you are
driving a VS Code-compatible client rather than writing a bespoke integration, prefer that surface
— it is the reason an unmodified VSCodium, Theia or Gitpod can repoint at open-vsx.org by changing
one URL. See `conformance/eclipse-conformance.yml`.

## Rules

- **Rate limits are signalled, not published.** Read `X-RateLimit-Remaining` on every response and
  back off before it hits zero. On a `429`, honour `Retry-After`. No numeric quota is documented.
  Note the header spelling: Open VSX uses `X-RateLimit-*`, while the Eclipse Foundation IT APIs use
  `X-Rate-Limit-*`. They are different headers.
- **Failures can arrive inside a 200.** Open VSX does not use the Eclipse `Error` envelope; a
  failed lookup may return the ordinary success schema with an `error` string field populated.
  Check that field before trusting a result.
- **Read-only skill.** Publishing is deliberately out of scope — see
  `eclipse-publish-to-open-vsx.md` for why writes need a human.
