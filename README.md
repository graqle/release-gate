# GraQle Release Gate

**Governance check for every PyPI and VS Code Marketplace release.**

A GitHub Action that runs a GraQle governance check on your release candidate. Blocks on `BLOCKER` findings. Warns on `MAJOR`. Produces a JSON report for audit.

Powered by the [graqle](https://pypi.org/project/graqle/) SDK.

---

## Why

Releases break things. A governance check catches:

- Trade secrets leaking into shipped code (internal identifiers, patent references, thresholds)
- Dependency drift (a transitive pulled in an unpatched vulnerability)
- Missing required files (`LICENSE`, `CHANGELOG.md`, version consistency)
- API regressions (renamed exports, removed public methods)
- Test coverage drops below threshold

Running this before PyPI publish or Marketplace publish prevents the "we noticed after release" incident.

## Quick start

Drop this into `.github/workflows/release-gate.yml`:

```yaml
name: Release Gate

on:
  release:
    types: [published]
  pull_request:
    paths:
      - 'pyproject.toml'
      - 'CHANGELOG.md'
      - 'src/**'

jobs:
  gate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # governance needs full history

      - uses: graqle/release-gate@v1
        with:
          target: pypi
          strict: 'true'
```

That's it. The Action installs `graqle` from PyPI and runs the gate. No other setup.

## Inputs

| Name | Required | Default | Description |
|------|----------|---------|-------------|
| `target` | yes | — | `pypi` or `vscode-marketplace` |
| `strict` | no | `true` | If `true`, BLOCKER findings fail the job |
| `confidence_threshold` | no | `0.65` | Minimum confidence (0.0–1.0) |
| `gate_threshold` | no | `0.60` | Minimum gate score (0.0–1.0) |
| `graqle_version` | no | *(latest)* | Pin a specific `graqle` version for determinism |
| `python_version` | no | `3.11` | Python runtime version |

## Outputs

| Name | Description |
|------|-------------|
| `verdict` | `CLEAR`, `WARN`, `FLAG`, or `INSUFFICIENT_GRAPH` |
| `confidence` | Float 0.0–1.0 |
| `report_path` | Path to the JSON report (also uploaded as artifact) |

## Usage patterns

### PyPI only

```yaml
- uses: graqle/release-gate@v1
  with:
    target: pypi
```

### VS Code Marketplace only

```yaml
- uses: graqle/release-gate@v1
  with:
    target: vscode-marketplace
```

### Both, in one workflow

```yaml
jobs:
  gate-pypi:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: graqle/release-gate@v1
        with:
          target: pypi

  gate-vscode:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: graqle/release-gate@v1
        with:
          target: vscode-marketplace
```

### Pin a specific graqle version

```yaml
- uses: graqle/release-gate@v1
  with:
    target: pypi
    graqle_version: '0.52.0'
```

### Read the verdict in a later step

```yaml
- id: gate
  uses: graqle/release-gate@v1
  with:
    target: pypi

- name: Block publish on non-CLEAR verdict
  if: steps.gate.outputs.verdict != 'CLEAR'
  run: exit 1
```

## Versioning

- `@v1` — floating pointer, latest stable v1.x. **Use this for most cases.**
- `@v1.0.0` — immutable, exact. Use if you need deterministic builds.
- `@main` — bleeding edge (not recommended).

Breaking input/output changes bump to `v2`. The governance logic itself lives in the `graqle` PyPI package and updates independently — re-running the Action always picks up the latest SDK.

## License

MIT — same as [graqle](https://github.com/quantamixsol/graqle).

## Getting help

- Issues: https://github.com/graqle/release-gate/issues
- SDK docs: https://pypi.org/project/graqle/
- Marketplace: https://marketplace.visualstudio.com/items?itemName=graqle.graqle
