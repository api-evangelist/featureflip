---
name: dead-flag-cleanup
description: Find confidently-dead or stale feature flags from real evaluation traffic and archive them safely, so code and dashboard stop disagreeing.
api: Featureflip Management API
base_url: https://api.featureflip.io
auth: "Authorization: Bearer <ffp_ personal or ffs_ service token>"
operations:
  - "GET /api/v1/orgs/{org}/projects/{project}/flags/removal-candidates"
  - "GET /api/v1/orgs/{org}/projects/{project}/flags/{flag}"
  - "POST /api/v1/orgs/{org}/projects/{project}/flags/{flag}/archive"
  - "POST /api/v1/orgs/{org}/projects/{project}/flags/{flag}/restore"
---

# Clean up dead feature flags

Detecting a dead flag is the easy half; removing it safely is the point. Archive
first (reversible), delete only after the code path is gone.

## Steps

1. **List removal candidates.**
   `GET .../flags/removal-candidates` returns confidently-dead flags. Pass
   `?staleness=stale` to widen it to stale-or-dead. Cursor-paginate via
   `next_cursor`.
2. **Inspect each candidate.** `GET .../flags/{flag}` to confirm the key, tags,
   and current variations before acting.
3. **Remove the code first.** Archiving removes the flag from the configuration
   SDKs receive, so every remaining call site immediately falls back to its
   default. Delete the code path before archiving, not after.
4. **Archive.** `POST .../flags/{flag}/archive`. This is reversible.
5. **Recover if needed.** `POST .../flags/{flag}/restore` restores a previously
   archived flag (no stated time window). Deleting a flag is permanent and a
   recreated flag gets a new bucketing salt.

## Rules

- The cleanup GitHub Action wraps this flow: it reads removal-candidates, opens
  one reviewable PR per flag, and archives on merge. Prefer it for source edits.
- Never delete a flag still referenced in code; archive, verify, then delete.
- Check the `error` code in the stable envelope (e.g. `forbidden` if the token
  role is below Member).
