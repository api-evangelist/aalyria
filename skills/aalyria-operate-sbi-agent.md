---
name: Operate a Spacetime SBI agent (control stream and telemetry)
description: Run a device-side Southbound Interface agent that holds the bidirectional Scheduling stream with the Spacetime controller, enacts beam/radio/flow control updates, reports per-request status, and pushes telemetry back.
api: grpc/api/scheduling/v1alpha/scheduling.proto
operations:
  - ReceiveRequests
  - Reset
  - ExportMetrics
generated: '2026-08-02'
method: generated
source: https://docs.spacetime.aalyria.com/api/sbi/
---

# Operate a Spacetime SBI agent

The Southbound Interface (SBI, also called CDPI — Control to Data-plane Interface) is how a
device participates in a Spacetime network. It is not a request/response API: it is one
long-lived bidirectional gRPC stream plus a metrics push. A reference Go agent ships in
`github.com/aalyria/api` under `agent/`.

## Preconditions

- `aalyria-connect-and-verify-instance` completed. The agent authenticates with the same
  self-signed RS256 JWT, audience-bound per method.
- The device is modelled — an `SdnAgent` and its `NetworkNode`/`NetworkInterface` entities
  exist in the model. The controller will not schedule work for a device it has no model of.

## The control stream

**`ReceiveRequests`** (`aalyria.spacetime.api.scheduling.v1alpha.Scheduling`) is
`stream ReceiveRequestsMessageToController` → `stream ReceiveRequestsMessageFromController`.

1. Open the stream and keep it open. Reconnection is expected, not exceptional.
2. Read `ControlPlaneUpdate` messages from the controller. These carry beam steering, radio
   configuration, and flow control changes, each with a validity time — an update is
   scheduled to be enacted at a moment, not immediately.
3. Enact the change on the device at the scheduled time.
4. Write back a response on the same stream carrying `request_id` and a
   `google.rpc.Status`. **`request_id` is the correlation key** — the controller matches
   your response to its request by that field alone.

Report failures **in the status field**, not by dropping the stream. Spacetime models
enactment outcome as data: `ControlPlaneUpdate` carries `scheduled`,
`enactment_attempted`, `unscheduled` and `completed` status fields precisely so a failed
physical action is visible to the solver and can be routed around.

**`Reset`** clears the controller's view of the agent's state. Call it after a device
restart or any event that makes previously acknowledged state untrue, so the controller
re-issues rather than assuming.

## Telemetry

**`ExportMetrics`** (`aalyria.spacetime.api.telemetry.v1alpha.Telemetry`) pushes measured
metrics and observations back to Spacetime. Returns `google.protobuf.Empty` — it is
fire-and-forget from the agent's point of view. This is what closes the loop: the solver's
link-budget model is corrected by what the radio actually measured.

## Failure handling

- `UNAVAILABLE` (14) — the stream dropped. Reconnect with backoff, then consider `Reset` if
  device state may have diverged while disconnected.
- `UNAUTHENTICATED` (16) — a long-lived stream outlives a short-lived JWT. Re-mint the token
  and re-establish; do not hold a stream open assuming the credential stays valid.
- A response with a non-OK `google.rpc.Status` is a normal, expected message — it tells the
  controller the enactment failed so it can re-solve. It is not a client error.

## Do not

- Do not enact a `ControlPlaneUpdate` early or late relative to its scheduled time; the
  whole system is temporospatial and a beam pointed at the wrong moment points at nothing.
- Do not silently drop an unactionable request. Answer it with a status.
- Do not open multiple concurrent `ReceiveRequests` streams for the same agent identity.
