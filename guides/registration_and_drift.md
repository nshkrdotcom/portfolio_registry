# Registration and drift

## What earns a row

A row is a repository, identified by its remote. Registration begins with a
canonical Git root whose Git common directory is its own `.git` and whose origin
is the recorded `owner/repository`. Worktrees, alternate clones, backups,
generated trees, and unrelated private trees are not rows.

**A repository does not have to build with Mix.** A Python, Rust, Go, or
JavaScript repository is registered on its identity alone and carries no `mix`
block. Language detection records what a repository carries; it never decides
whether the repository belongs.

The conventional checkout of a repository is its remote's name under the
operator's checkout root. A repository checked out elsewhere is bound through
the operator's binding file. That file is machine-local and is never committed
here — which is what lets a portfolio spread across several checkout roots stay
describable by one portable catalog.

## Mix metadata

Where a repository does build with Mix, discovery evaluates `mix.exs` in an
isolated process without loading `config/runtime.exs`, starting the application,
compiling project code, or checking dependencies. Generated dependency trees,
`_legacy` archives, backups, worktrees, and vendored source are pruned before
evaluation.

A project whose metadata cannot be read enters the dated unresolved ledger. It
is never guessed from a filename.

Two projects providing one application is a recorded fact, not a failure. The
catalog lists both; a dependency declaration that must choose names a
`provider`. What was once an unresolved shadow copy is now ordinary identity.

## Absence

Absence from the catalog and absence from disk are different questions, and
neither is evidence about the other.

- A repository present on the remote and absent from the catalog is drift. It is
  either registered or given a `disposition` that says why it is not tracked.
- A repository in the catalog and absent from disk is a catalog entry the
  operator has not checked out. The catalog is complete without it, and the
  entry is not drift. **The tooling does not yet operate around it:** binding
  stops at the first absent checkout, so every command that binds — `doctor`,
  `inventory`, `plan`, `run` — fails until the checkout exists or a binding
  entry points at it. Sparse binding, which classifies each repository as
  bound, absent, or invalid and continues past an absent one, is the next unit
  of work in Mix Workspace Ops. Do not write an operation against the
  behaviour until it lands.
- A checkout on disk that matches no catalog row and no operator ignore entry is
  drift, and the drift gate fails on it.

The operator ignore ledger is machine-local: worktrees, scratch clones,
generated checkouts, experiments. Nothing about one operator's disk belongs in
the catalog, so the ledger never moves here.

## Validation

Validation is external. The catalog has no `mix.exs`, no executable, no runtime
dependency, and no Hex release; Mix Workspace Ops loads and validates it.

Validation checks structure, vocabulary, and the rules that need the whole
document at once: project ids unique across the catalog, every repository in at
least one group, no group carried by every repository, every dependency
declaration that can resolve locally having exactly one provider or naming it,
every release-chain package provided by a project of its own repository, and
every declared prerequisite present.

Binding additionally verifies the Git root, common directory, origin, and the
explicit checkout root supplied at run time.

## Snapshots

Snapshots under `snapshots/` preserve dated observations and migration receipts.
They are evidence. Nothing consults them as dependency or compatibility policy,
and nothing rewrites an earlier one when a later observation disagrees.
