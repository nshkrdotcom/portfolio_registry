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

Machine-readable identities, classifications, dependency sources, views, and
release ordering for the complete `nshkrdotcom` portfolio.

This repository is data, not build machinery. It has no `mix.exs`, executable,
runtime dependency, or Hex release. Generic consumers such as Mix Workspace Ops
load it explicitly, validate it from outside, and bind its portable GitHub
identities to operator-owned checkouts.

## The unit is a repository

A repository is the unit of the catalog, whatever it is written in. Every
repository carries its remote identity, languages, lifecycle, disposition,
visibility, roles, groups, and agent scope. Mix projects are an optional block
inside a repository record, so a repository that builds nothing with Mix is a
complete row that views select and operations act on.

Repositories that consume cross-repository applications also carry the table
that says how each one resolves, and the packages they publish as one release
train.

The catalog records **180 repositories** across five toolchains. 724 Mix
projects appear inside the repositories that build with Mix; 715 applications
are provided across them, two of them by more than one project. Fourteen groups
partition and overlap the portfolio; every repository carries at least one, and
none carries them all.

## Authority

The catalog owns:

- stable repository identity, remote coordinate, and default branch;
- classification: languages, lifecycle, disposition, visibility, roles, groups,
  agent scope;
- optional Mix project identity, path, kind, and the applications each provides;
- how each consumed application resolves — candidate sources, order, publish
  order, Mix options, and provider selection where more than one project
  provides it;
- which packages publish as one train, and the ordering edges derivation cannot
  see;
- named views over the one inventory;
- dated drift and migration evidence.

Each project's `mix.exs` remains authoritative for dependencies, requirements,
version, and package contents. The catalog holds a resolution table, never a
dependency list.

The catalog holds portable remote facts only. It carries no absolute path, no
relative sibling path, no operator directory name, no credential, and no
executable code. A repository checked out somewhere unconventional is bound
through an operator-owned, machine-local binding file that is never committed
here.

## Layout

- `registry.json` — `portfolio_registry.registry/v2`; one row per repository.
- `views/all.json` — every repository.
- `views/nshkr.json` — the NSHKR platform, selected by group.
- `views/dependency_sources.json` — repositories carrying an installed
  dependency-source helper.
- `views/release_train.json` — repositories publishing as one release train.
- `views/python.json` — repositories carrying a Python toolchain.
- `snapshots/` — dated observations and migration receipts; evidence only.
- `guides/` — contract, view, and drift rules.

## Validate and bind

Build Mix Workspace Ops from its independent source repository, then run:

```bash
mix_workspace_ops registry validate \
  --registry /path/to/portfolio_registry/registry.json

mix_workspace_ops registry select \
  --registry /path/to/portfolio_registry/registry.json \
  --view /path/to/portfolio_registry/views/nshkr.json

mix_workspace_ops registry chain \
  --registry /path/to/portfolio_registry/registry.json \
  --package cli_subprocess_core

mix_workspace_ops doctor \
  --registry /path/to/portfolio_registry/registry.json \
  --checkout-root /path/to/operator/checkouts
```

Bindings are machine-local and untracked. A normal checkout uses the
repository's remote name under the supplied checkout root; anything else needs
an explicit operator-owned binding file.

## Schema versions

`portfolio_registry.registry/v2` and `portfolio_registry.view/v2` are current.
`mix_workspace_ops.registry/v1` and `mix_workspace_ops.view/v1` still load and
are normalized onto the same records. Acceptance of the v1 schemas is scheduled
to be removed after **2027-02-23**.

See [Registry contract](guides/registry_contract.md),
[Views](guides/views.md), and
[Registration and drift](guides/registration_and_drift.md).
