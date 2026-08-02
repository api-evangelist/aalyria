---
name: Provision traffic-engineering intent and inspect the computed solution
description: Declare SR-TE policies, links, downtimes and constraint groups through the Spacetime Provisioning API, then read back the beams and candidate paths the Spacetime solver actually computed through the Solution API.
api: grpc/api/provisioning/v1alpha/provisioning.proto
operations:
  - CreateLink
  - ListLinks
  - CreateP2pSrTePolicy
  - ListP2pSrTePolicies
  - CreateP2pSrTePolicyCandidatePath
  - CreateDowntime
  - ListDowntimes
  - QueryBeams
  - GetBeam
  - QueryP2pSrTePolicyCandidatePaths
generated: '2026-08-02'
method: generated
source: https://docs.spacetime.aalyria.com/api/nbi/
---

# Provision traffic-engineering intent and inspect the computed solution

Provisioning is **intent**, not configuration. You declare what the network should achieve;
Spacetime's solver decides which links to form, which beams to steer and which paths to
use, continuously, as platforms move. The Solution API is how you read that decision back.

Two different endpoints: `provisioning-v1alpha.$DOMAIN` for intent,
and the Solution service for solver output.

## Preconditions

- `aalyria-connect-and-verify-instance` completed.
- `aalyria-build-network-model` completed — every provisioning object references model
  entities that must already exist.

## Declare intent (Provisioning)

- **`CreateLink`** / **`ListLinks`** — declare a link between two modelled interfaces.
- **`CreateP2pSrTePolicy`** / **`ListP2pSrTePolicies`** — a point-to-point segment-routing
  traffic-engineering policy: the unit of "carry this traffic from A to B under these
  constraints". `CreateP2mpSrTePolicy` is the point-to-multipoint form.
- **`CreateP2pSrTePolicyCandidatePath`** — candidate paths hang off a policy; a policy with
  no candidate path expresses a goal with no way to meet it.
- **`CreateDowntime`** / **`ListDowntimes`** — planned unavailability of an asset, so the
  solver routes around it ahead of time instead of reacting to a failure.
- **`CreateProtectionAssociationGroup`** and **`CreateDisjointAssociationGroup`** — group
  policies that must be protected together, or must not share fate. Both select their
  members through a `subjects_filter` string rather than an explicit id list.
- **`CreateGeographicRegion`** — a named region other objects constrain against.

Updates take an `update_mask` (`google.protobuf.FieldMask`) — send the changed fields only.
Batch and `Purge*` variants exist on the Solution mutation surface for bulk changes.

## Read the solution

- **`QueryBeams`** — which beams the solver computed, with their validity intervals.
- **`GetBeam`** — one beam by id.
- **`QueryP2pSrTePolicyCandidatePaths`** — which candidate paths were selected and when.

Because Spacetime is temporospatial, results are **time-bounded**: a beam or path is valid
over an interval, not "currently true". Always read the interval, and re-query rather than
caching a solution across time.

## Declarative alternative

```
nbictl provisioning-v1alpha sync -r --dry-run <dir>
nbictl provisioning-v1alpha sync -r <dir>
nbictl provisioning-v1alpha list
```

`--max-concurrency` (default 100) throttles in-flight requests; Spacetime publishes no
rate-limit policy, so lower it if a large sync starts failing `UNAVAILABLE`.

## Failure handling

- `FAILED_PRECONDITION` (9) — a referenced model entity, link, or parent policy does not
  exist yet. Create it first.
- `NOT_FOUND` (5) — confirm ids with the matching `List*` RPC before retrying.
- Empty `QueryBeams` result is **not** an error. It usually means the intent is
  unsatisfiable over the requested interval (no access window, a `Downtime` covering the
  asset, or a disjointness constraint that cannot be met) — check the constraints before
  assuming a fault.

## Do not

- Do not treat provisioning writes as idempotent; no idempotency key is published. Use
  `--dry-run` sync or read back with the `List*` RPC to confirm before retrying.
- Do not write to the Solution API to "fix" a route. The solver owns it; change the intent.
