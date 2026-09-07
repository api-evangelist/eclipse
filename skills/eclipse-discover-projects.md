---
name: eclipse-discover-projects
description: Look up Eclipse Foundation projects, their lifecycle phase, licences, source repositories and release history using the Eclipse Projects PMI API. Use when asked what an Eclipse project is, who maintains it, what it is licensed under, or what it has released.
api: Eclipse Projects PMI API
base_url: https://projects.eclipse.org
spec: openapi/eclipse-projects-pmi-api-openapi.yml
operations:
  - FetchProjects
  - RetrieveAProject
  - FetchProjectReleases
  - FetchProjectReviews
  - FetchGithubReposByProject
  - FetchProposals
  - FetchInterestGroups
auth: none
---

# Discover Eclipse Foundation projects

The Projects PMI (Project Management Infrastructure) API is the Eclipse Foundation's
machine-readable record of every project it hosts. It needs no authentication and no account.

## Steps

1. **Find the project.** `FetchProjects` — `GET /api/projects`. Returns the project list with
   metadata. Filter with the `page` and `pagesize` query parameters; the response paginates by
   RFC 8288 `Link` header, so follow `rel="next"` rather than incrementing blindly.

2. **Resolve the identifier.** Every project is keyed by `project_id`, a dotted path such as
   `technology.openvsx` or `ecd.theia`. This identifier is the join key across the whole Eclipse
   estate — the Working Groups, Git ECA and Project Adopters APIs all accept it, though none of
   them declares a `$ref` to a shared schema (see `data-model/eclipse-data-model.yml`).

3. **Read the project.** `RetrieveAProject` — `GET /api/projects/{project_id}`. Returns licences,
   `github_repos`, `github`/`gitlab` info, committers, and the project's lifecycle phase under the
   Eclipse Project Handbook (Incubation, Mature, Archived).

4. **Read its releases.** `FetchProjectReleases` — `GET /api/projects/{project_id}/releases`.
   `FetchProjectReviews` — `GET /api/projects/{project_id}/reviews` — returns the formal release
   and graduation reviews, which is how you confirm a phase change actually happened.

5. **Find its source.** `FetchGithubReposByProject` — `GET /api/github/{project_id}`.

## Rules

- **Read-only.** This API publishes no write operation. Nothing here can change Eclipse state.
- **Two operations are deprecated.** `FetchSimultaneousReleases` (`GET /api/simultaneous_release`)
  and `RetrieveSimultaneousRelease` are flagged `deprecated: true` in the specification with no
  stated replacement. Do not build on them.
- **Errors.** Only `200` and `404` are declared. A `404` means the `project_id` does not exist;
  there is no "exists but hidden" distinction. Failures carry the Eclipse `Error` envelope
  (`status_code`, `message`, `url`) — not RFC 9457 problem+json. See
  `errors/eclipse-problem-types.yml`.
- **No rate limit is published.** Budget conservatively; no quota, window or 429 is documented for
  this host. See `rate-limits/eclipse-rate-limits.yml`.
- **No change feed.** There is no webhook or event stream for project changes. If you need to
  track a project over time you must poll. See `asyncapi/eclipse-webhooks.yml`.
