# h2c-manager

![vibe coded](https://img.shields.io/badge/vibe-coded-ff69b4)
![python 3](https://img.shields.io/badge/python-3-3776AB)
![heresy: 1/10](https://img.shields.io/badge/heresy-1%2F10-brightgreen)
![stdlib only](https://img.shields.io/badge/dependencies-stdlib%20only-brightgreen)
![public domain](https://img.shields.io/badge/license-public%20domain-brightgreen)

Lightweight package manager for [helmfile2compose](https://github.com/helmfile2compose/helmfile2compose). Downloads a distribution (full or bare engine) and CRD operator modules from GitHub releases. Python 3, stdlib only — no dependencies.

## Usage

```bash
# Download from main (rolling release)
curl -fsSL https://raw.githubusercontent.com/helmfile2compose/h2c-manager/main/h2c-manager.py -o h2c-manager.py

# Distribution only (default: helmfile2compose)
python3 h2c-manager.py

# Distribution + operators (from CLI)
python3 h2c-manager.py keycloak cert-manager trust-manager servicemonitor

# Distribution + operators (from helmfile2compose.yaml depends list)
python3 h2c-manager.py

# Pin versions
python3 h2c-manager.py --distribution-version v2.1.0 keycloak==0.2.0

# Use the bare engine instead of the full distribution
python3 h2c-manager.py --distribution core

# Extensions only (no distribution)
python3 h2c-manager.py --no-distribution keycloak

# Custom install directory
python3 h2c-manager.py -d ./tools keycloak

# Skip download, reuse cached .h2c/ (missing files still downloaded)
python3 h2c-manager.py --no-reinstall

# Run helmfile2compose with smart defaults
python3 h2c-manager.py run -e compose
```

## Run mode

`run` is a shortcut to invoke helmfile2compose with sensible defaults:

```bash
python3 h2c-manager.py run -e compose
# equivalent to:
# python3 .h2c/helmfile2compose.py --helmfile-dir . --extensions-dir .h2c/extensions --output-dir . -e compose
```

By default, `run` re-downloads everything before each invocation (latest or pinned version from `helmfile2compose.yaml`). Use `--no-reinstall` to skip already-cached files. There is no version tracking — a cached file is either kept or replaced with whatever version resolves.

Defaults: `--helmfile-dir .`, `--extensions-dir .h2c/extensions` (if it exists), `--output-dir .`. Any explicit flag overrides the default. All extra arguments are passed through to helmfile2compose.

## Declarative dependencies

If `helmfile2compose.yaml` exists, h2c-manager reads `distribution`, `distribution_version` and `depends` from it:

```yaml
distribution: helmfile2compose   # optional (default: helmfile2compose)
distribution_version: v2.1.0     # optional (latest if omitted)
depends:
  - keycloak
  - cert-manager==0.1.0
  - trust-manager
```

```bash
python3 h2c-manager.py
# Distribution version from helmfile2compose.yaml: v2.1.0
# Reading extensions from helmfile2compose.yaml: keycloak, cert-manager==0.1.0, trust-manager
```

CLI flags (`--distribution-version`, `--distribution`, explicit extension args) override the yaml.

Backwards compatibility: `core_version` in yaml is read as a fallback for `distribution_version`.

## Output

```
.h2c/
├── helmfile2compose.py          # or h2c.py if distribution: core
└── extensions/
    ├── keycloak.py
    ├── cert_manager.py          # auto-resolved as dep of trust-manager
    └── servicemonitor.py
```

## Distributions

`distributions.json` maps distribution names to GitHub repos. Two distributions are available:

| Name | Script | Description |
|------|--------|-------------|
| `helmfile2compose` | `helmfile2compose.py` | Full distribution — core + built-in extensions |
| `core` | `h2c.py` | Bare engine — no built-in converters |

### Schema

```json
{
  "schema_version": 1,
  "distributions": {
    "<name>": {
      "repo": "<org>/<repo>",
      "file": "<filename>.py",
      "description": "human-readable description"
    }
  }
}
```

## Extension registry

`extensions.json` maps extension names (operators, ingress rewriters, etc.) to GitHub repos. Available versions are whatever tags/releases exist on each repo — the registry doesn't list versions, GitHub is the source of truth.

### Schema

```json
{
  "schema_version": 1,
  "extensions": {
    "<name>": {
      "repo": "<org>/<repo>",
      "description": "human-readable description",
      "file": "<filename>.py",
      "depends": ["<other-extension>"]
    }
  }
}
```

### Adding an extension

Open a PR to this repo adding your extension to `extensions.json`. The repo must have at least one GitHub Release.

## Documentation

Full docs: [h2c-manager usage guide](https://helmfile2compose.github.io/maintainer/h2c-manager/)

## License

Public domain.
