---
name: targeting-rule
description: Serve a specific variation to a targeted audience using a reusable segment and a first-match-wins targeting rule, per environment.
api: Featureflip Management API
base_url: https://api.featureflip.io
auth: "Authorization: Bearer <ffp_ personal or ffs_ service token>"
operations:
  - "POST /api/v1/orgs/{org}/projects/{project}/segments"
  - "GET /api/v1/orgs/{org}/projects/{project}/flags/{flag}/environments/{env}/targeting"
  - "POST /api/v1/orgs/{org}/projects/{project}/flags/{flag}/environments/{env}/rules"
  - "PUT /api/v1/orgs/{org}/projects/{project}/flags/{flag}/environments/{env}/rules/reorder"
---

# Target a segment with a rule

Serve one variation to users who match an attribute-based rule (or a reusable
segment), and the fallthrough default to everyone else. Rules evaluate top to
bottom, first match wins.

## Steps

1. **Create a reusable segment (optional).**
   `POST .../segments` with a flat condition list (operators: equals, contains,
   startsWith, endsWith, in) joined by a single AND or OR. A segment built with
   AND across two disjoint groups matches nobody by construction.
2. **Read current targeting.**
   `GET .../flags/{flag}/environments/{env}/targeting` to see the enabled state,
   existing rules, and the fallthrough.
3. **Add a rule.**
   `POST .../flags/{flag}/environments/{env}/rules` binding the variation to the
   segment or an inline attribute condition. Send an `Idempotency-Key`.
4. **Order the rules.**
   `PUT .../rules/reorder` — order is part of the logic, since the first matching
   rule wins and stops evaluation.

## Rules

- Editing rules, weights, or conditions does NOT re-bucket users (bucketing uses
  a per-flag salt + user key). Changing the bucket-by attribute does.
- An attribute missing from the evaluation context makes its rule fail to match;
  a type/case mismatch does too.
- Targeting is per environment — a rule in staging changes nothing in production.
