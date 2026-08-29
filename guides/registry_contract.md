# Registry contract

`registry.json` implements `portfolio_registry.registry/v2`.

A **repository** is the unit of the catalog. Mix projects are an optional detail
inside a repository record, not the unit of anything. A repository that builds
nothing with Mix — a Python analysis tree, a Rust extension, a documentation
site — is a complete catalog row with no `mix` block at all.

`mix_workspace_ops.registry/v1` still loads for one minor series of Mix
Workspace Ops. See [Schema versions](#schema-versions).

**Every example below is checked.** A block with a `schema` key is a whole
document and is valid as it stands. Every other block is a repository record
keyed by `id`, carrying the fields its passage is about; together they assemble
into one document, and `mix_workspace_ops registry examples --guide
guides/registry_contract.md` refuses the guide if that document does not
validate. An example that could not be catalogued is a defect in the guide.

## The document

The document has exactly two keys and no others. `schema` names the version;
`repositories` is the list of repository records. A document carrying any
further key is refused, so a field added at the top level is a mistake caught
at validation rather than data silently ignored.

```json
{
  "schema": "portfolio_registry.registry/v2",
  "repositories": [
    {
      "id": "shape_analysis",
      "github": "example-org/shape-analysis",
      "default_branch": "main",
      "languages": ["python"],
      "lifecycle": "active",
      "disposition": "tracked",
      "visibility": "public",
      "roles": ["library"],
      "groups": ["estate.example", "family.analysis"],
      "agent_scope": "eligible"
    }
  ]
}
```

That document is complete and valid. Everything below describes what a
repository record may carry beyond the required fields.

## A repository record

Every repository carries these fields. None is optional.

| Field | Meaning |
|---|---|
| `id` | Stable lowercase identity. Never reused, never renamed to follow a remote rename |
| `github` | `owner/repository` |
| `default_branch` | The remote's default branch |
| `languages` | Non-empty list of toolchains the repository carries. A repository may carry several |
| `lifecycle` | `active`, `maintenance`, `dormant`, or `archived` |
| `disposition` | `tracked`, `superseded`, `archived`, or `intentionally_untracked` |
| `visibility` | `public` or `private` |
| `roles` | What the repository is for. May be empty |
| `groups` | Non-empty list of groups an operation can target |
| `agent_scope` | `eligible`, `restricted`, or `never` |

`lifecycle` describes activity; `disposition` describes the intent behind
tracking the remote. They are separate: a maintained repository you have stopped
tracking is `active` and `intentionally_untracked`, and a superseded one whose
remote is still receiving commits is `active` and `superseded`.

`disposition` describes the **remote**. A machine-local observation — a
worktree, a scratch clone, a generated checkout, a directory you keep off the
drift report — belongs in the operator's local ignore ledger, not here.

Every repository carries at least one group, and no group is carried by every
repository. A group that selects everything selects nothing useful, and the
validator refuses one.

The record inside that document is complete as it stands, and it builds nothing
with Mix:

```json
{
  "id": "shape_analysis",
  "github": "example-org/shape-analysis",
  "default_branch": "main",
  "languages": ["python"],
  "lifecycle": "active",
  "disposition": "tracked",
  "visibility": "public",
  "roles": ["library"],
  "groups": ["estate.example", "family.analysis"],
  "agent_scope": "eligible"
}
```

## The optional `mix` block

A repository that builds with Mix adds a `mix` block listing its projects, and a
`workspace` entry when its projects form one workspace. `shape` is the running
example from here on:

```json
{
  "id": "shape",
  "github": "example-org/shape",
  "default_branch": "main",
  "languages": ["elixir"],
  "lifecycle": "active",
  "disposition": "tracked",
  "visibility": "public",
  "roles": ["library"],
  "groups": ["family.shape"],
  "agent_scope": "eligible",
  "mix": {
    "projects": [
      { "id": "shape.shape_workspace", "app": null, "path": ".",
        "kind": "workspace_root" },
      { "id": "shape.shape_runtime", "app": "shape_runtime",
        "path": "core/shape_runtime", "kind": "package" },
      { "id": "shape.shape_rpc", "app": "shape_rpc",
        "path": "protocols/shape_rpc", "kind": "package" },
      { "id": "shape.vendored_shape_core", "app": "shape_core",
        "path": "vendor/shape_core", "kind": "package" },
      { "id": "shape.shape_consumer", "app": "shape_consumer",
        "path": "packaging/consumer", "kind": "generated" }
    ]
  }
}
```

A project record:

| Field | Meaning |
|---|---|
| `id` | Stable project identity, unique across the whole catalog |
| `app` | The OTP application the project builds, or `null` |
| `path` | Repository-relative path. Never absolute, never escaping the repository, never entering `.git`, `deps`, `_build`, or operator state |
| `kind` | `standalone`, `workspace_root`, `package`, `tooling`, or `generated` |
| `provides` | Applications this project supplies for dependency resolution. Defaults to `[app]`; must contain `app` when `app` is not `null` |
| `current` | Optional temporal mark selecting this project when several projects provide the same application; at most one per application |
| `lineage` | Optional project id this project descends from; documentation only, never read by resolution |

`generated` marks a Mix project that is build output rather than source — a
projected consumer tree, for instance, as `shape.shape_consumer` is. Nothing
derives workspace membership or seeds a dependency closure from a generated
project. Derivation cannot tell that a directory is generated, so the catalog
says so.

Real Mix projects beneath `examples/` are inventory, not noise. Their presence
does not turn a standalone root into a workspace or change any stable id.

### Application identity is not provider selection

`provides` states identity. It is entirely normal for more than one project to
provide one application: a fork, an example, a successor, a copy vendored into a
monorepo. The catalog records all of them. `shape.vendored_shape_core` above is
a vendored copy of an application another repository also provides:

```json
{
  "id": "shape_core",
  "github": "example-org/shape-core",
  "default_branch": "main",
  "languages": ["elixir"],
  "lifecycle": "active",
  "disposition": "tracked",
  "visibility": "public",
  "roles": ["library"],
  "groups": ["family.core"],
  "agent_scope": "eligible",
  "mix": {
    "projects": [
      { "id": "shape_core.shape_core", "app": "shape_core",
        "path": "core/shape_core", "kind": "package" }
    ]
  }
}
```

Two projects now provide `shape_core`, and that is legal identity, not an error.
Choosing between them is separate and follows one order: a declaration's
`provider`; a provider in the consumer's own repository; the single provider
marked `current`; otherwise an error naming every candidate and repository.
Nothing silently takes the first match.

`current` is temporal and load-bearing: when a successor takes over an
application name, the mark moves. It is unrelated to repository
`lifecycle: "active"`, which says only that the repository is alive. `lineage`
may state that a project descends from another, but it is documentation and
never changes resolution.

### Workspace membership

Membership has one authority: derivation. The catalog says which mechanism to
derive from, and records members only as exceptions.

```json
{
  "id": "shape",
  "mix": { "workspace": { "kind": "blitz" } }
}
```

`kind` is `umbrella` or `blitz`. It names the mechanism the repository builds
with; it does not change how membership derives.

Derivation reads the project metadata in the record itself, and applies one
rule to both kinds:

- the project of kind `workspace_root` is the container the workspace is rooted
  at, not a member of it;
- a project of kind `generated` is build output, so it is never a member and
  cannot be made one;
- every other project in the repository is a member.

`include_project_ids` and `exclude_project_ids` are the exceptions. The common
one is a Blitz root whose own project globs name `"."`, so the root builds as
part of its own workspace; derivation cannot read those globs, so the record
says so:

```json
{
  "id": "shape",
  "mix": {
    "workspace": {
      "kind": "blitz",
      "include_project_ids": ["shape.shape_workspace"]
    }
  }
}
```

A Mix umbrella never needs it: an umbrella root is a container by definition. A
repository whose projects are unrelated omits `workspace` entirely.

`mix_workspace_ops registry workspace --registry registry.json` reports the
derived members of every workspace, and `--repository ID` reports one.

## Dependency sources

`dependency_sources` maps an application name to how it can be resolved. It is a
**resolution table, not a dependency list**: each project's `mix.exs` remains the
authority for which dependencies exist and what versions they require. An entry
for an application a project does not declare is simply never used.

```json
{
  "id": "shape",
  "dependency_sources": {
    "shape_core": {
      "github": {},
      "hex": "~> 0.2.0",
      "provider": "shape_core.shape_core",
      "opts": { "override": true }
    },
    "sprite": {
      "github": { "repo": "other-org/sprite-ex", "branch": "main" },
      "order": ["github"],
      "publish_order": ["github"]
    }
  }
}
```

| Field | Meaning |
|---|---|
| `github` | `{}` or `true` opts into coordinates derived from provider identity; `repo`, `branch`, `ref`, `tag`, `subdir` carry only deviations |
| `hex` | A requirement string, or `{ "requirement": "...", "package": "..." }` for a differently named published fork |
| `order` | Sources tried, in order. Omitted in the common case, when it is `["local", "github", "hex"]` |
| `publish_order` | Sources tried while publishing. Omitted in the common case, when it is `["hex"]` |
| `provider` | Project id supplying the application, where more than one does |
| `opts` | Mix dependency options merged into the emitted tuple: `override`, `runtime`, `optional`, `only`, `targets` |

Both entries in that table show the rule the section turns on. `shape_core`
inherits the default order, names its durable provider deviation, and opts into
GitHub without restating that provider's repository, branch, or project path.
`sprite` is a third-party application
no catalog project provides, so its order omits `local` rather than pointing at
a provider that does not exist.

**`local` carries no path.** It resolves to the provider project's `path` inside
the operator's checkout of that project's repository. That is the whole reason
paths are absent: a relative sibling path is a fact about one operator's disk,
while the provider's repository and project path are portable. Where a
repository is checked out somewhere unconventional, the operator's binding file
says so, and it is never committed here.

An omitted `order` falls through a source the entry does not carry, exactly as
resolution has always worked; it only has to be able to reach something. An
order written out states intent, so every source it names must be declared, and
`local` must have a resolvable provider. That is what makes an application with
no catalog project impossible to leave half-declared: either catalogue it, or
write an order that omits `local`.

`opts` are load-bearing. `override: true` decides which version wins a diamond;
dropping it changes what Mix resolves.

All three sources use one provider identity. Local derives the bound checkout
and project path. GitHub derives the provider repository, default branch, and
project path as `subdir`; its block exists to opt in and carries only
deviations. Hex derives package name from application name and always carries a
requirement; the map form declares the exceptional package name and is emitted
through Mix's `hex:` option. The catalog never carries a local path.

A project may carry its own `dependency_sources` block. Its entries replace the
repository's entry for those applications alone, which is how one project in a
repository can require a different version of something than its siblings.

## Release chain

`release_chain` declares which of a repository's packages publish as one train,
and the prerequisite edges derivation cannot see.

```json
{
  "id": "shape",
  "release_chain": {
    "shape_runtime": [],
    "shape_rpc": ["shape_runtime"]
  }
}
```

A package with an entry is in the train; the value lists prerequisites beyond
the derived ones. Derivation supplies every cross-repository edge from the
dependency-source tables. It cannot supply an edge between two packages of the
same repository, because a repository-scoped table does not say which project
consumes an entry — so that edge is declared, as `shape_rpc`'s is above, or the
consuming project declares its own table and restores the attribution.

Empty lists are meaningful: they put a package in the train with no prerequisite
of its own.

## What the catalog never holds

Dependency edges, version requirements a `mix.exs` already states, package
versions, package contents, absolute paths, relative sibling paths, credentials,
CI commands, operator directory names, and executable code. Those belong to
current Mix metadata, to the projection tooling, or to machine-local operator
state.

## Schema versions

`portfolio_registry.registry/v2` is current. `mix_workspace_ops.registry/v1`
still loads and is normalized onto the same records: one language, `elixir`;
every repository `active`, `tracked`, and `public` with `agent_scope` `eligible`
and no roles; project tags become repository groups; each project provides its
own `app`; and no dependency sources or release chain. Acceptance of v1 is
scheduled to be removed after **2027-02-23**.
