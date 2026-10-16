# orbit-auth development

```bash
python -m pip install -e ../orbit-core
python -m pip install -e '.[dev]'
pip-audit --skip-editable --progress-spinner off
pytest
ruff check src tests
ruff format --check src tests
mypy
python -m build
```

Keep vendor SDKs and provider transports in the relevant `orbit-auth-*` integration. Add tests for
credential redaction, invalid and boundary input, cancellation, and deny-by-default behavior.
Credential-bearing live tests must be opt-in and use dedicated non-production identities.

The current local quality matrix passes on CPython 3.11, 3.12, 3.13, and 3.14: all 103 tests,
Ruff lint and format checks, strict mypy, dependency audits, and wheel/sdist builds pass on each
interpreter. Editable Core is skipped by pip-audit and is audited in its own repository. These are
local results; hosted CI and provider interoperability remain separate evidence.
