# orbit-auth operations

## Configure authentication explicitly

Basic authentication is disabled unless `BasicAuthenticator` is passed to the ASGI application.
Create each static credential with `BasicCredential.create(username, password, roles=...)`, passing
the password from protected configuration as `SecretStr` where practical, then pass the resulting
credential models in a sequence to `BasicAuthenticator`. Credential creation derives a salted
PBKDF2-HMAC-SHA256 digest; the model retains the random salt and digest, not the plaintext password.
There are no default accounts or passwords.

Credential creation is intended for trusted bootstrap/configuration work. Keep the source password
in a secret manager, restrict who can change the configured credential set, and rotate credentials
through the deployment's secret/configuration process. Do not generate credentials from request
data or store plaintext passwords in application databases, configuration files, logs, or images.
All configured users must use the same PBKDF2 work factor so login cost does not disclose which
account record was selected.

Use HTTPS end to end or a trusted TLS terminator. Configure Core's trusted forwarded-header policy
before using proxy-provided client information. Do not trust arbitrary forwarding headers.
Basic authentication sends a reusable password with every request, so enforce TLS at the public
edge and ensure the request scheme reflects a trusted TLS termination. The default authenticator
rejects non-HTTPS requests. Setting `require_tls=False` is appropriate only for isolated local
development and should not be enabled for an internet-facing service.

The authenticator performs PBKDF2 verification in worker threads with a per-instance concurrency
bound. Keep the configured credential set small and the concurrent verification limit appropriate
for available CPU; a full attacker queue can still consume CPU. Compose it with a rate limiter and
network-level controls. This static authenticator does not provide user lifecycle, MFA, federation,
password reset, session cookies, or distributed account state.

## Token and provider behavior

Bearer authentication requires a verifier implementation. Use `orbit-auth-jwt` only with explicit
algorithm, issuer, audience, and key configuration. Use `orbit-auth-oauth2` with a shared,
short-lived transaction store and a trusted session binding. Do not log tokens, authorization
codes, state values, PKCE verifiers, credentials, or raw provider responses.

The built-in token revocation store is process-local and is lost on restart. Multi-worker or
distributed deployments need a separately implemented shared store. This repository does not claim
to provide provider discovery transport, JWKS fetching, account recovery, MFA, or identity lifecycle.
