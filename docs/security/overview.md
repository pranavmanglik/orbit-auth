# orbit-auth security

Authentication is opt-in. There are no default accounts, default passwords, or plaintext password
stores. Protect Basic Auth with TLS, secret management, rotation, and network controls; it is not a
complete enterprise identity system.

The Basic authenticator accepts only explicitly constructed salted PBKDF2 hashes, uses a generic
failure for malformed, unknown, and incorrect credentials, and runs a bounded number of password
checks off the event loop. It defaults to requiring HTTPS. It does not persist users, throttle
requests across processes, or provide account lockout; deploy rate limiting and network controls
for exposed login routes. Basic credentials are reusable on every request, so use a stronger
session or federated authentication flow when the application requires those properties.

Bearer token verification is injected. Do not accept unverified token text as identity. Configure
allowed algorithms, issuer, audience, clock policy, and key rotation explicitly. The optional JWT
integration owns cryptographic verification. The optional OAuth integration requires a secure
transaction store, atomic state consumption, PKCE, and browser-session binding for callback flows.
Verifier exceptions become a generic authentication failure with their exception chain suppressed;
adapter diagnostics must still avoid logging credentials, token contents, or provider responses.

Keep tokens, password material, authorization codes, state, nonces, and provider responses out of
logs, traces, exceptions, and support bundles. Review effective permissions and identity-provider
settings in each deployment. The package is a v0.1.0-alpha review candidate; hosted CI and provider
interoperability remain open, and it has no production certification.
