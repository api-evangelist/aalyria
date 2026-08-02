---
name: Connect to a Spacetime instance and verify credentials
description: Establish authenticated gRPC access to an Aalyria Spacetime instance using a self-signed RS256 JWT, then confirm the connection with the Status API before attempting any model or provisioning change.
api: grpc/api/status/v1/status.proto
operations:
  - GetVersion
  - GetMetrics
generated: '2026-08-02'
method: generated
source: https://docs.spacetime.aalyria.com/api/authentication/
---

# Connect to a Spacetime instance and verify credentials

Do this first. Every other Spacetime skill assumes a working credential and a reachable
instance. Spacetime is deployed **per customer**, so there is no shared production host —
you need the `$DOMAIN` Aalyria issued for the instance.

## Preconditions

- An RSA keypair, generated locally. `nbictl generate-keys --org "<ORG>"` writes the
  `.key` and `.crt` to `~/.config/nbictl/keys` by default.
- `USER_ID`, `KEY_ID` and `DOMAIN`, returned by Aalyria after you send them the **public**
  `.crt` only. The private `.key` must never be sent to anyone, including Aalyria.

## Authentication contract

Spacetime does **not** use OAuth. Each call carries a JWT the client signs itself:

- header: `alg: RS256`, `kid: $KEY_ID`, `typ: JWT`
- payload: `iss: $USER_ID`, `sub: $USER_ID`, `exp`, `iat`, and
  `aud: https://${DOMAIN}/${GRPC_SERVICE}/${GRPC_METHOD}`

The audience is **per-RPC**. A token minted for one method will not authenticate another.
Mint a fresh token per method, or use `nbictl generate-auth-token --audience <aud>
--expiration 1h`, which defaults to a one-hour validity.

Send the token as a bearer token in the `Authorization` gRPC metadata header.

## Endpoint resolution

The default strategy is `subdomain`: each API is reached at `<api>-<version>.$DOMAIN`
on port 443 — `model-v1.$DOMAIN`, `provisioning-v1alpha.$DOMAIN`, `status-v1.$DOMAIN`.
`single_domain` and `custom` strategies exist; with `custom` you set `--model_url`,
`--provisioning_url`, `--status_url` and a `--default_url` individually. Do not include
a scheme (`https://`, `dns:///`) or an API prefix in `--url`.

## Steps

1. Configure a profile:
   `nbictl config set --url <DOMAIN> --user_id <USER_ID> --key_id <KEY_ID> --priv_key <PATH>`
   Use `--profile <name>` to keep multiple instances separate.
2. Call **`GetVersion`** (`aalyria.spacetime.api.status.v1.StatusService`) —
   `nbictl status-v1 get-version`. A semantic version string back means TLS, the JWT
   signature, the `kid`, and the audience are all correct.
3. Call **`GetMetrics`** — `nbictl status-v1 get-metrics` — to confirm the instance is
   actually processing traffic. It returns OpenTelemetry `Gauge`/`Sum` messages:
   `platform_types`, `scheduling_inbound_count`, `scheduling_outbound_count`.

## Failure handling

- `UNAUTHENTICATED` (16) — the token expired, the `kid` does not match the enrolled key,
  or the `aud` does not match the exact service/method being called. Re-mint the token
  for that specific method before retrying.
- `PERMISSION_DENIED` (7) — credential is valid but not authorized. Check with
  `CheckPermission` on `aalyria.spacetime.api.permissions.v1alpha.Permissions`.
- `UNAVAILABLE` (14) — wrong `$DOMAIN` or subdomain, or the instance is down. There is no
  public status page; `GetVersion` is the health check.

## Do not

- Do not retry a `UNAUTHENTICATED` failure with the same token — the audience binding will
  fail identically.
- Do not use `--auth_strategy=none` or `--transport_security=insecure` against anything
  other than a local test instance.
