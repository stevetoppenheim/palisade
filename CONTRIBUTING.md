# Contributing to Palisade

Thanks for your interest. A few ground rules keep this project safe and maintainable.

## Principles

1. **Authorization is sacred.** Any change that could let Palisade act on a target
   outside the configured scope is rejected on sight. The scope enforcer
   (`palisade/scope.py`) is default-deny and must stay that way; new features route
   through it, never around it.
2. **Detection, not exploitation.** Palisade finds and evidences vulnerabilities. We do
   not accept automated exploitation, credential brute-forcing, payload delivery, or
   self-propagation. Verification stays human-driven and scope-locked.
3. **Claims must match code.** Don't document a capability the code doesn't implement.
   If it's planned, put it under "Roadmap," not "Features."

## Development setup

```bash
python3 -m pip install -r requirements.txt pytest
python3 -m pytest -q            # unit + parser tests
python3 -m palisade.cli selfcheck
```

## Adding a tool/step

1. Add a wrapper function in `palisade/tools.py` returning `list[Finding]`.
2. Register it in `TOOL_STEPS` and `STEP_TOOLS`.
3. Add a parser test in `tests/test_tool_parsers.py` using a **realistic** sample of the
   tool's actual output (see the existing ones). Parsers must fail loudly, not silently.
4. Update `config/pipeline.example.yaml` and the README feature list.

## Pull requests

- Keep the test suite green; add tests for new behavior.
- Run `selfcheck` and include its result if you touched config schemas.
- Reference any related audit finding ID (e.g. `F-01`) in the PR description.
