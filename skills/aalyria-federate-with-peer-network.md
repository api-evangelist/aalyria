---
name: Federate capacity with a peer network operator
description: Use the Spacetime Federation (East-West) API to discover a peer operator's transceivers and contact windows, request interconnection through bearers and attachment circuits, and advertise or consume transponded satellite capacity.
api: grpc/api/federation/interconnect/v1alpha/interconnect.proto
operations:
  - ListTargets
  - GetTarget
  - ListCompatibleTransceiverTypes
  - ListTransceivers
  - GetTransceiver
  - ListContactWindows
  - CreateBearer
  - ListBearers
  - GetBearer
  - DeleteBearer
  - CreateAttachmentCircuit
  - ListAttachmentCircuits
  - GetAttachmentCircuit
  - DeleteAttachmentCircuit
  - ListTransponders
  - ListChannels
  - CreateChannel
generated: '2026-08-02'
method: generated
source: https://docs.spacetime.aalyria.com/api/federation/
---

# Federate capacity with a peer network operator

The Federation API is the East-West interface: it lets two *different* operators request and
supply capacity from each other, so one can fill a coverage gap and the other can sell
otherwise-idle assets. Note the namespace — `outernet.federation.interconnect.v1alpha` and
`outernet.federation.transponder.v1alpha`. This is a community-namespaced inter-operator
interface, not an Aalyria-proprietary one, which is what makes it safe to implement against
a peer who is not an Aalyria customer.

## Preconditions

- `aalyria-connect-and-verify-instance` completed against your own instance.
- A commercial/peering relationship with the counterpart operator. The API expresses the
  mechanics of interconnection; it does not create the agreement.

## Interconnect: discover, then request

1. **`ListTargets`** / **`GetTarget`** — the endpoints the peer is willing to interconnect
   with. Start here; everything else is scoped to a target.
2. **`ListCompatibleTransceiverTypes`** — which of your transceiver types can actually close
   a link with theirs. Do this before proposing anything; incompatible RF is the most common
   dead end.
3. **`ListTransceivers`** / **`GetTransceiver`** — the concrete radios available.
   `CreateTransceiver`, `UpdateTransceiver` and `DeleteTransceiver` manage the ones you
   expose to the peer.
4. **`ListContactWindows`** — when the two assets can actually see each other. Because both
   sides may be in motion, capacity exists only inside a window; a request outside one
   cannot be met.
5. **`CreateBearer`** — request the physical/RF layer connection over a contact window.
   `ListBearers` / `GetBearer` / `DeleteBearer` manage its lifecycle.
6. **`CreateAttachmentCircuit`** — the logical circuit carried over the bearer.
   `ListAttachmentCircuits` / `GetAttachmentCircuit` / `DeleteAttachmentCircuit` manage it.

Order matters: target → compatibility → transceiver → contact window → bearer → attachment
circuit. Skipping a layer fails `FAILED_PRECONDITION` rather than auto-creating the layer
beneath.

## Transponder: advertise or consume bent-pipe capacity

The transponder service (`outernet.federation.transponder.v1alpha`) covers transponded
satellite payloads:

- **`ListSatellites`** / **`GetSatellite`**, **`ListTransponders`** / **`GetTransponder`** —
  the assets.
- **`ListChannels`** / **`GetChannel`** — the capacity units on a transponder.
- **`CreateChannel`**, **`UpdateChannel`**, **`DeleteChannel`** — allocate, retune or release
  a channel.
- **`ListTransmitBeams`** / **`GetTransmitBeam`**, **`ListReceiveBeams`** /
  **`GetReceiveBeam`** — the beams a channel rides on.

## Failure handling

- `PERMISSION_DENIED` (7) — cross-operator authorization is enforced per target. Verify with
  `CheckPermission` on the Permissions service; the authorization config is versioned, so
  `ListAuthorizationConfigRevisions` shows when it changed.
- Empty `ListContactWindows` — no geometry, not a fault. Widen the requested interval or
  choose a different transceiver pair.
- `FAILED_PRECONDITION` (9) — a lower layer (bearer, transceiver) does not exist yet.

## Do not

- Do not retry `CreateBearer` or `CreateAttachmentCircuit` blindly after an ambiguous
  failure. There is no idempotency key; list first and reconcile, or you may double-book
  capacity you will be billed for.
- Do not cache a contact window or a channel allocation across time — both are interval-valid.
