# Orbit Auth

`orbit-auth` is Orbit's provider-neutral authentication and authorization capability. It provides
opt-in Basic and bearer authentication, principal and token models, token validation and revocation
contracts, JWKS models, role and policy authorization, and OAuth/OIDC contracts. Core keeps only
generic ASGI extension protocols and request-context handling; it does not provide these
authentication mechanisms.

```bash
python -m pip install orbit-core orbit-auth
```

Basic authentication uses salted PBKDF2 password hashes, has no default credentials, requires TLS
by default, and never stores plaintext passwords. It is disabled unless an application explicitly
configures it:

```python
from orbit import Application, ApplicationConfig
from orbit.asgi import ASGIApplication
from orbit_auth import BasicAuthenticator, BasicCredential, RoleAuthorizer
from pydantic import SecretStr

application = Application(ApplicationConfig(name="admin-service"))
admin_credential = BasicCredential.create(
    "operator",
    SecretStr(admin_password_from_secret_manager),
    roles=("orbit.admin",),
)
authenticator = BasicAuthenticator([admin_credential])
asgi = ASGIApplication(
    application,
    authenticator=authenticator,
    authorizer=RoleAuthorizer(),
)
```

Provider and protocol integrations remain separately installable:

| Package | Responsibility |
|---|---|
| `orbit-auth-oauth2` | OAuth 2.0 authorization-code flow and PKCE |
| `orbit-auth-jwt` | JWT verification using PyJWT |
| `orbit-auth-rbac` | Exact role-to-permission evaluation |

They depend on `orbit-auth` contracts and own their optional behavior and dependencies. `orbit-security`
is a separate package for cross-cutting controls such as in-process HTTP rate limiting; it does not
own authenticators, principals, or token contracts.

## Security and production use

Never place credentials in source control or use example values as real passwords. Load credentials
from a managed secret source, use TLS, configure trusted proxy handling before relying on forwarded
client addresses, and define credential rotation and recovery outside this package. Basic Auth is
intended for explicit development and simple administration scenarios; it does not provide account
lifecycle, MFA, federation, password reset, or a persistent identity store. Token revocation is
process-local unless the application supplies a shared store. JWKS types are contracts, not a key
fetch or rotation service.

This is a **v0.1.0-alpha review candidate**. On CPython 3.11.14, 3.12.14, 3.13.15, and 3.14.7,
all 103 tests, Ruff lint and format checks, strict mypy, dependency audits, and wheel/sdist builds
pass locally. Hosted CI and provider-specific identity interoperability remain release gates. The
alpha status is not a production certification or a substitute for threat modeling and
identity-provider validation.

## Documentation and development

Read the [architecture](docs/architecture/overview.md), [operations](docs/operations/README.md),
[security](docs/security/overview.md), and [development](docs/development/README.md) guides.

```bash
python -m pip install -e '.[dev]'
pytest
ruff check src tests
ruff format --check src tests
mypy
python -m build
```

Licensed under Apache-2.0.
