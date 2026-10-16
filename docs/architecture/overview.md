# Orbit Auth architecture

`orbit-auth` is an optional capability package for authentication and authorization in Orbit
applications. It implements Core's public ASGI authenticator and route-authorizer contracts while
leaving request dispatch, application lifecycle, and request context ownership in Core.

```text
Orbit Core: generic ASGI authentication hooks and request context
└── orbit-auth: authentication mechanisms, principals, token and policy contracts
    ├── orbit-auth-oauth2: OAuth 2.0 authorization-code flow and PKCE
    ├── orbit-auth-jwt: JWT verification using PyJWT
    └── orbit-auth-rbac: exact role-to-permission evaluation

orbit-security: cross-cutting controls such as in-process HTTP rate limiting
```

The provider packages are separately installable. Core does not install this package, and this
package does not load integrations or credentials implicitly. The dependency direction is:

```text
application ──> orbit-core
application ──> orbit-auth ──> orbit-core public contracts
application ──> optional provider adapter ──> orbit-auth contracts
```

## Implemented responsibilities

- **Basic authentication** validates a bounded UTF-8 credential header, compares users using a
  salted PBKDF2-HMAC-SHA-256 hash, and returns an immutable principal. It is explicitly configured,
  requires HTTPS by default, and has no built-in users or password persistence.
- **Bearer authentication** delegates token verification to an injected verifier contract. The
  bearer parser does not itself validate JWT signatures or interpret provider-specific claims.
- **Identity and token models** validate and freeze bounded claims, roles, scopes, times, and
  provider identifiers. These models represent verified input only when constructed by a trusted
  authenticator/verifier; constructing a model does not verify an external credential.
- **JWKS models** hold a bounded, time-limited key snapshot. They do not fetch keys, validate
  signatures, enforce algorithm policy, or implement refresh and rotation.
- **Authorization** supplies exact role matching and a named in-process policy registry. Unknown
  policies deny access. Resource access is granted only when the application registers and invokes
  the relevant evaluator at the enforcement point.
- **OAuth/OIDC contracts** describe provider-neutral authorization data. Network transport,
  redirect handling, PKCE execution, state/nonce persistence, token exchange, and provider failures
  belong to the corresponding adapter where implemented.

## Trust and enforcement boundaries

Authentication proves only the claims that the configured authenticator or verifier validates.
Authorization is a separate check: a valid principal is not automatically permitted to use a
route or access a resource. Applications must attach required roles or invoke a named policy at
every relevant enforcement point, including background jobs and non-HTTP entry points.

Basic Auth credentials are held in a static in-process map. Rotating them requires rebuilding or
replacing the authenticator, and multiple workers each have their own configured copy. Token
revocation is an interface/policy concern and is process-local unless an installed implementation
uses a shared durable store. Neither behavior is a distributed session system.

The Basic password work factor and concurrent verification limit bound individual authenticator
work. Configure a small, reviewed credential set; do not treat this mechanism as an account
directory or a general-purpose identity provider. Ensure Core's trusted-proxy configuration is
correct before using forwarded scheme or client address information in security decisions.

## Extension rules

Keep vendor SDKs, secrets, network clients, credential discovery, key retrieval, and provider error
translation in provider adapters. Keep portable principals, token policy, authorization hooks, and
validation contracts in `orbit-auth`. Cross-cutting controls such as a request rate limiter belong
to `orbit-security`, rather than being coupled to a particular authentication provider.

The Python package is the capability API. An implementation in another language should be a
separate package only when it has a concrete workload benefit and a documented conformance boundary;
it should not introduce a second, subtly different contract.

## Stability

The package is a v0.1.0-alpha review candidate; public contracts may change before a stable release.
The local Python 3.11–3.14 test, lint, type, and build matrix passes. Hosted CI and provider-specific
identity interoperability remain open release gates. Pin compatible versions across Core,
capability, and provider adapters, and validate the complete composition in the application's own CI.
