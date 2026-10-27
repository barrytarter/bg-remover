# Workflow - bg-remover

Check-only profile. It records how changes to this repository are verified; it does not
grant authority to merge, release or alter repository settings.

## Project

- C++ command-line tool (`src/bg-remover.cpp`) built with the `Makefile` and per-platform
  Dockerfiles (`Dockerfile.ubuntu`, `Dockerfile.ubuntu-arm64`); macOS builds natively.
- There is no unit-test suite in the repository.

## Hard gate

The pull request's own CI is the gate. There is no separate local full suite.

- Workflow: `.github/workflows/build.yml` ("Build Multi-Platform Binaries").
- Triggers: `pull_request` targeting `main`, `v*` tag pushes, and `workflow_dispatch`.
- Required checks (matrix-generated names): `Build ubuntu`, `Build linux-arm64`,
  `Build macos`. Each builds the platform binary and verifies it and its checksum.
- A missing or red build check is a hold, not a waived gate.

## Release

- The `release` job runs only for tags or manual dispatch and publishes the GitHub release.
- Release, tag and dispatch operations are maintainer-only and are never part of a
  workflow-change pull request.

## Runner selection

- `runs-on` for the `build` job defaults to the `runner` value in each matrix entry.
- For same-repository pull requests only, the `ubuntu` matrix entry may be redirected by
  the repository variable `CI_RUNNER`. Fork pull requests, tag/manual runs, `linux-arm64`
  and `macos` always use their matrix runner.

## Local checks for workflow edits

- Parse changed YAML (`python -c "import yaml,sys;[yaml.safe_load(open(f)) for f in sys.argv[1:]]" <files>`).
- Run `actionlint` on changed workflow files.
- Compare against `origin/main` and confirm only the intended lines changed.
- Do not run Docker builds locally for workflow-only changes; CI covers them.

## Contribution notes

- Open pull requests as drafts; the maintainers decide readiness and merging.
- Keep matrix keys, job ids and check names stable, since they identify required checks.
