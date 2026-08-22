# Aalyria

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
