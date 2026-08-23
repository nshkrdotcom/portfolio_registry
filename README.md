<p align="center">
  <img src="assets/portfolio_registry.svg" width="200" alt="Portfolio Registry logo" />
</p>

<p align="center">
  <a href="https://github.com/nshkrdotcom/portfolio_registry">
    <img alt="GitHub: nshkrdotcom/portfolio_registry" src="https://img.shields.io/badge/GitHub-nshkrdotcom%2Fportfolio__registry-0b0f14?logo=github" />
  </a>
  <a href="https://github.com/nshkrdotcom/portfolio_registry/blob/main/LICENSE">
    <img alt="License: MIT" src="https://img.shields.io/badge/license-MIT-0b0f14.svg" />
  </a>
</p>

# Portfolio Registry

Canonical machine-readable identities, classifications, views, and operational
profiles for the complete `nshkrdotcom` Elixir ecosystem.

This repository is data, not build machinery. It has no `mix.exs`, executable,
runtime dependency, dependency-edge policy, package-version policy, or Hex
release. Generic consumers such as Mix Workspace Ops load it explicitly and bind
its portable GitHub identities to operator-owned checkouts.

The 2026-08-11 zero-default baseline records 162 repositories and 695 Mix
projects. Of those, 688 have unique application identities and seven are valid
non-application workspace roots. The NSHKR view selects 416 projects from 43
classified repositories; the global view selects all 695. Ambiguous projects
remain dated evidence instead of receiving guessed identities.

## Authority

The registry owns only:

- stable repository and Mix-project identities;
- GitHub coordinates and relative project paths;
- application identity used to map current Mix dependencies, when the project
  is an application;
- project classification tags and operational-profile references;
- named views over the one global inventory;
- dated drift and migration evidence.

Each project's `mix.exs` remains authoritative for dependencies, requirements,
version, and package contents. Projected poncho packages keep their projection
metadata in their owning repository. The registry never duplicates either.

## Layout

- `registry.json` — one canonical row per repository and Mix project.
- `views/all.json` — the entire resolved registry.
- `views/nshkr.json` — a tag-selected NSHKR platform subset, not a copied list.
- `profiles/operator_profiles.json` — shared operational classifications.
- `snapshots/` — dated observations and unresolved identities; evidence only.
- `guides/` — protocol, view, and drift rules.

## Validate and bind

Build Mix Workspace Ops from its independent source repository, then run:

```bash
mix_workspace_ops registry validate \
  --registry /path/to/portfolio_registry/registry.json

mix_workspace_ops registry select \
  --registry /path/to/portfolio_registry/registry.json \
  --view /path/to/portfolio_registry/views/nshkr.json

mix_workspace_ops doctor \
  --registry /path/to/portfolio_registry/registry.json \
  --checkout-root /path/to/operator/checkouts
```

Bindings are machine-local and untracked. A normal checkout uses the repository
basename under the supplied checkout root. Exceptional layouts require an
explicit operator-owned binding file; they are never encoded in this registry.

The initial snapshot records the portable migration surface for 51 copied
dependency-source helpers and 50 adjacent configs across 41 canonical
repositories. It stores repository identities, relative paths, commits, status,
and content digests—never checkout paths, credentials, or dependency policy.
The zero-default baseline snapshot records the later example-project and
non-application-root reconciliation without rewriting that initial evidence.

See [Registry contract](guides/registry_contract.md),
[Views](guides/views.md), and
[Registration and drift](guides/registration_and_drift.md).
