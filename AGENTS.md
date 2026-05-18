# Repository Guidelines

## Project Structure & Module Organization

This repository is a Kubernetes workspace with `one-mount-ai-platform/` checked in as a git submodule. Root manifests such as `dummy-pod.yaml`, `run-pod/`, and `volume/` are for cluster experiments.

Inside `one-mount-ai-platform/`, the Python monorepo is a `uv` workspace:

- `apps/`: product applications, currently `apps/fraud_detection/`.
- `sdks/`: shared platform SDKs such as `auth`, `dc_client`, `datastore`, `inference`, `mlops`, and `observability`.
- `infra/`: Kubernetes and infrastructure configuration.
- `docs/`: architecture notes, ADRs, and runbooks.
- `tools/`: repo automation and CI/release helpers.

Package source usually lives under `src/<package>/`; MLOps packages live under `sdks/mlops/packages/*/`.

## Build, Test, and Development Commands

Run Python workspace commands from `one-mount-ai-platform/`:

- `uv sync --all-packages`: install all workspace packages.
- `uv run pytest`: run tests under `sdks/*/tests` and `apps/*/tests`.
- `uv run ruff check .`: lint Python code.
- `uv run mypy .`: type-check with strict mypy settings.
- `uv run --package fraud-detection fraud-detection`: run the fraud detection app entrypoint.

For root Kubernetes manifests, validate before applying: `kubectl apply --dry-run=client -f dummy-pod.yaml`.

## Coding Style & Naming Conventions

Use Python 3.12 for the main platform workspace unless a package states otherwise; `sdks/mlops` supports Python 3.11. Ruff is configured for 100-character lines, import sorting, pyupgrade, bugbear, and related rules. Mypy is strict by default.

Use `snake_case` for Python modules, functions, and package directories. Keep app-specific logic in `apps/*`; shared platform behavior belongs in `sdks/*` and requires owner review.

## Testing Guidelines

Tests use `pytest` with `test_*.py` or `*_test.py` naming. Place SDK tests in `sdks/<name>/tests/` and app tests under `apps/<app>/tests/`. New SDK functions need unit tests; new app endpoints need integration coverage. Respect package coverage gates, such as 85% for `sdks/dc_client` and 60% for `apps/fraud_detection`.

## Commit & Pull Request Guidelines

Recent commits use short imperative summaries, for example `fix job` or `Add PostGres Cluster Config On Bare Metals`. Keep commits focused and mention the affected area when useful. SDK releases must use `release(<sdk>): vX.Y.Z`.

Pull requests should describe the change, list tests run, link issues when available, and call out affected owners. SDK changes require Core Platform review; `auth` and `datastore` also require Security review. Include CHANGELOG and version updates for SDK releases.

## Security & Configuration Tips

Never commit secrets, credentials, kubeconfigs, or private keys. Use `onemount-auth`, `onemount-datastore`, and other platform SDKs instead of direct one-off implementations for auth, storage, observability, or tenant-aware data access.

## Kubernetes

When working with Kubernetes manifests, Helm charts, or Kustomize overlays, follow the workflow in `.kubernetes-skill/SKILL.md`.
Load references from `.kubernetes-skill/references/` as needed.
