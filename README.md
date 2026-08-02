# Aalyria

Aalyria Technologies builds software and hardware for operating resilient, high-throughput
networks in motion across space, air, land and sea. Its flagship platform, **Spacetime**, is
a temporospatial software-defined networking system that models the time-dynamic position
and orientation of satellites, aircraft, ships and ground stations, then continuously plans,
schedules and steers the links between them. Aalyria also produces **Tightbeam**, a
free-space optical communications system. The company was spun out of Alphabet in 2022,
carrying technology developed for Project Loon and Taara.

- Website: https://www.aalyria.com/
- Developer docs: https://docs.spacetime.aalyria.com/
- API source: https://github.com/aalyria/api (Apache-2.0)

## The contract

Spacetime has **no OpenAPI, no GraphQL and no REST surface**. The machine-readable contract
is proto3 + gRPC, published openly under Apache-2.0: **45 `.proto` files, 13 services,
182 RPCs, 512 messages**, harvested verbatim into [`grpc/`](grpc/) with the upstream `api/`
layout preserved so import paths resolve.

Three interfaces:

| Interface | What it does |
|---|---|
| **Northbound (NBI)** | Define and orchestrate the network — Model (NMTS entities/relationships), Provisioning (SR-TE policies, links, downtimes, constraint groups), Solution (computed beams and paths), Simulation, Permissions, Audit |
| **Southbound (SBI / CDPI)** | Device-facing control plane — a long-lived bidirectional `ReceiveRequests` stream carrying beam/radio/flow control updates, plus `ExportMetrics` telemetry push |
| **Federation (East-West)** | Inter-operator interconnection and transponded capacity, in the community `outernet.federation.*` namespace rather than Aalyria's own |

Instances are deployed **per customer** on a customer-specific `$DOMAIN`, resolved by
subdomain (`model-v1.$DOMAIN`, `provisioning-v1alpha.$DOMAIN`, `status-v1.$DOMAIN`).
Authentication is a **self-signed RS256 JWT** whose `aud` claim is bound to a single gRPC
service *and method* — there is no OAuth and no shared production host.

## Artifacts

| Directory | Type | Method |
|---|---|---|
| [`grpc/`](grpc/) | Protobuf (45 files + index) | searched |
| [`authentication/`](authentication/) | Authentication | searched |
| [`packages/`](packages/) | Packages / SDKs (Go, Python) | searched |
| [`cli/`](cli/) | CLI (`nbictl`) | searched |
| [`changelog/`](changelog/) | ChangeLog (20 releases) | searched |
| [`lifecycle/`](lifecycle/) | Lifecycle + stability levels + deprecation policy | searched |
| [`conformance/`](conformance/) | Conformance (18 standards) | derived |
| [`conventions/`](conventions/) | Conventions | derived |
| [`errors/`](errors/) | ErrorCatalog (gRPC status model) | derived |
| [`data-model/`](data-model/) | DataModel | derived |
| [`mcp/`](mcp/) | MCPServer — **candidate only**, 182 tools mapped 1:1 to real RPCs | derived |
| [`skills/`](skills/) | AgentSkill × 5 | generated |
| [`llms/`](llms/) | LLMsTxt | generated |
| [`security/`](security/) | DomainSecurity | probed |

## Notable absences (probed 2026-08-02, all missing)

`llms.txt`, `/.well-known/security.txt`, `/.well-known/agent-card.json`,
`/.well-known/agent.json`, OIDC/OAuth discovery, `/.well-known/api-catalog`, OpenAPI,
AsyncAPI, a public status page, public pricing, self-serve sign-up, a hosted MCP server, a
Postman collection, a vulnerability disclosure policy, and a trust center. Spacetime is sold
and deployed under commercial agreement, so credentials are issued out of band. Instance
health is exposed programmatically instead, through `StatusService.GetVersion` /
`GetMetrics` (OpenTelemetry format).

Spacetime also publishes **no idempotency contract** — no idempotency key, no client token,
no request-deduplication guarantee on any write RPC. The `nbictl ... sync` flow is a
client-side convergence loop, not a server-side guarantee.
