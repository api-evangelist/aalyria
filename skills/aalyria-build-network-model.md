---
name: Build and maintain the Spacetime network model
description: Create, read, update and delete the NMTS entities and relationships that make up a Spacetime digital twin through the Model API v1, including atomic fragment upserts and declarative directory sync.
api: grpc/api/model/v1/model.proto
operations:
  - CreateEntity
  - UpdateEntity
  - DeleteEntity
  - GetEntity
  - ListEntities
  - CreateRelationship
  - DeleteRelationship
  - ListRelationships
  - UpsertFragment
generated: '2026-08-02'
method: generated
source: https://docs.spacetime.aalyria.com/api/nbi/
---

# Build and maintain the Spacetime network model

The Model API (`aalyria.spacetime.api.model.v1.Model`) is the digital twin: a graph of
**NMTS entities** (platforms, antennas, interfaces, transceivers — typed by the Outernet
Council Network Model for Temporospatial Systems) joined by **typed relationships**.
Everything the Provisioning, Solution and SBI surfaces do is expressed against this graph,
so model first.

## Preconditions

Complete `aalyria-connect-and-verify-instance` first. Endpoint is `model-v1.$DOMAIN`.

## Read before you write

- **`ListEntities`** — takes a `filter` string. Note there is **no** `page_token` on this
  RPC; it returns the full matching set, so filter narrowly on large models.
- **`ListRelationships`** — same shape, for the edges.
- **`GetEntity`** — fetch one entity by id when you already know it.

## Write path

- **`CreateEntity`** returns the created `nmts.v1.Entity`.
- **`UpdateEntity`** — partial update. Send only the changed fields plus an update mask;
  do not read-modify-write the whole entity.
- **`CreateRelationship`** / **`DeleteRelationship`** — edges are created and removed
  independently of their endpoints.
- **`DeleteEntity`** — delete the edges that reference an entity before deleting it, or the
  call will fail `FAILED_PRECONDITION`.
- **`UpsertFragment`** — the preferred bulk path. A fragment carries a set of entities and
  relationships and is applied together, so a partially-built topology is never visible.

## Declarative alternative

For anything larger than a handful of objects, keep the model in `.textproto` files under
version control and reconcile:

```
nbictl model-v1 sync -r --dry-run <dir>    # trial run, no changes
nbictl model-v1 sync -r <dir>              # apply
nbictl model-v1 sync -r --delete <dir>     # also remove remote objects absent locally
```

This is a client-side convergence loop, not a server-side transaction. Always run
`--dry-run` first, and treat `--delete` as destructive.

## Ordering rule

Create referenced entities before the relationships that point at them, and before any
provisioning object (`CreateLink`, `CreateP2pSrTePolicy`) that names them. Spacetime will
reject a dangling reference rather than create a placeholder.

## Failure handling

- `ALREADY_EXISTS` (6) — use `UpdateEntity` or `UpsertFragment` instead of `CreateEntity`.
- `FAILED_PRECONDITION` (9) — a referenced entity or relationship is missing, or an edge
  still points at the entity you are deleting.
- `INVALID_ARGUMENT` (3) — the message does not validate against the published proto.
  Retrying unchanged will fail identically.

## Do not

- Do not assume writes are idempotent. Spacetime publishes no idempotency key and no
  request-deduplication contract; a retried `CreateEntity` may create a second object or
  fail `ALREADY_EXISTS`. Read back with `GetEntity` before retrying a write whose outcome
  is unknown.
- Do not use `aalyria.spacetime.api.model.v0` for new work — v0 is superseded by v1.
