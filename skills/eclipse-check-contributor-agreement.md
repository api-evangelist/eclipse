---
name: eclipse-check-contributor-agreement
description: Check whether a contributor has a signed Eclipse Contributor Agreement (ECA) on file, and validate a set of commits against Eclipse contribution policy. Use when reviewing a contribution to an Eclipse project or diagnosing a failing ECA check on a pull request.
api: Eclipse Foundation Git ECA API
base_url: https://api.eclipse.org/git
spec: openapi/eclipse-git-eca-api-openapi.yml
operations:
  - getUserStatus
  - validate
  - getCommitValidation
  - getCommitValidationUI
  - getLatestContributionForUser
auth: none
---

# Check Eclipse Contributor Agreement status

The Git ECA API is the callable form of Eclipse contribution policy. It is what the Foundation's
own GitHub and GitLab checks call, and it is anonymous — you can diagnose a failing ECA check
without credentials.

## Steps

1. **Look up the contributor.** `getUserStatus` — `GET /eca/lookup`. Returns whether the account
   has a signed ECA on file. This answers the common question ("why is the ECA check red?")
   directly and is the cheapest call in the flow.

2. **Validate commits.** `validate` — `POST /eca` with the commit set. Returns per-commit
   validation, which is what the forge check displays. This is a *validation* call: it computes a
   verdict, it does not change Eclipse state, so it is safe to run repeatedly.

3. **Read a stored verdict.** `getCommitValidation` — `GET /eca/status/{fingerprint}` — returns a
   previously computed result by fingerprint. `getCommitValidationUI` —
   `GET /eca/status/{fingerprint}/ui` — returns the human-readable form to link a reviewer to.

4. **Check contribution history.** `getLatestContributionForUser` —
   `GET /eca/contributions/{username}/latest-contribution`.

## Rules

- **Do not call the webhook receivers.** `processGithubWebhook`, `processGitlabHook` and
  `revalidateWebhookRequest` are inbound endpoints for GitHub and GitLab to push into, not
  operations for a client to invoke. See `asyncapi/eclipse-webhooks.yml`.
- **`validate` is the only POST here that is effectively read-only** — it computes and returns a
  verdict. Everything else in this skill is a GET.
- **Errors.** `400` malformed commit payload · `401` / `403` on the reporting endpoints ·
  `404` unknown user or fingerprint · `500`. Eclipse `Error` envelope, not problem+json.
- **Remediation is human.** If the ECA is unsigned, the fix is for the contributor to sign it at
  <https://accounts.eclipse.org/> — there is no API operation that signs an agreement.
