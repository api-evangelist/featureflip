---
name: progressive-rollout
description: Create a boolean feature flag and gradually ramp it from 1% to 100% of users using a deterministic percentage rollout, per environment.
api: Featureflip Management API
base_url: https://api.featureflip.io
auth: "Authorization: Bearer <ffp_ personal or ffs_ service token>"
operations:
  - "POST /api/v1/orgs/{org}/projects/{project}/flags"
  - "PUT /api/v1/orgs/{org}/projects/{project}/flags/{flag}/environments/{env}"
  - "POST /api/v1/orgs/{org}/projects/{project}/flags/{flag}/environments/{env}/toggle"
  - "GET /api/v1/orgs/{org}/projects/{project}/flags/{flag}/environments/{env}"
---

# Progressive rollout of a feature flag

Ramp a feature to a growing share of users without redeploying. Bucketing is
deterministic (sticky), so raising the percentage keeps existing users in place
and only adds new ones.

## Steps

1. **Create the flag.** `POST /api/v1/orgs/{org}/projects/{project}/flags` with
   the flag key, name, and its boolean variations. Send a unique
   `Idempotency-Key` header so a retry does not create a duplicate.
2. **Enable it in the target environment.**
   `POST .../flags/{flag}/environments/{env}/toggle` to turn the flag on for that
   environment (a new environment defaults every flag to off).
3. **Set the rollout percentage.**
   `PUT .../flags/{flag}/environments/{env}` to set the serving strategy to a
   percentage rollout, starting at 1%. Bucketing needs a stable user key in the
   evaluation context — without one a server SDK serves the control variation.
4. **Ramp.** Re-`PUT` the same endpoint raising the percentage in steps
   (1 → 5 → 25 → 100). Lowering it removes the highest buckets; raising it back
   restores exactly the same users.
5. **Verify.** `GET .../flags/{flag}/environments/{env}` to confirm the current
   strategy and percentage.

## Rules

- Every mutating call returns the stable error envelope; check `error` (e.g.
  `validation_failed`, `forbidden`) not the HTTP status alone.
- The change streams to connected SDKs live over SSE within seconds.
- Reversible: toggle the flag off (kill switch) at any time; SDKs fall back to
  the default passed at the call site.
