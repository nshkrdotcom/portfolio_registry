# Views

Views implement `portfolio_registry.view/v2`. They are selectors, not manifests.
A view never copies a repository or project row; it states what to match, and
the catalog stays the single source of the rows.

A view selects **repositories first and projects second**. That ordering is the
point: a repository that builds nothing with Mix contributes no project, and a
project-only selector could never reach it. A repository-scoped operation — fetch
every remote, check every checkout for uncommitted work, report every lifecycle —
selects repositories and ignores projects entirely.

## Selector

Every field is optional. An omitted field constrains nothing.

| Field | Matches |
|---|---|
| `groups_any` | Repositories carrying at least one of these groups |
| `groups_all` | Repositories carrying all of these groups |
| `languages` | Repositories carrying at least one of these languages |
| `lifecycles` | Repositories whose lifecycle is one of these |
| `repository_ids` | Only these repositories |
| `exclude_repository_ids` | Everything except these repositories |
| `project_ids` | Within the selected repositories, only these projects |
| `exclude_project_ids` | Within the selected repositories, everything except these projects |

An empty selector selects the whole catalog:

```json
{
  "schema": "portfolio_registry.view/v2",
  "id": "all",
  "description": "Every repository in the portfolio, whatever it is written in.",
  "selector": {}
}
```

A selector naming an unknown repository or project id is refused rather than
silently matching nothing, so a renamed identity fails loudly.

## The views here

| View | Selects |
|---|---|
| `all` | Every repository |
| `nshkr` | `platform.nshkr` |
| `dependency_sources` | Every repository carrying an installed dependency-source helper |
| `release_train` | The active repositories providing packages that publish as one train |
| `python` | Repositories carrying a Python toolchain, whether or not they also build with Mix |

## Groups

Groups exist to be targeted. A group no operation would ever select should not
exist, which is why the v1 tags `canonical` and `ecosystem` — carried by all 695
projects — are gone.

| Group | Selects |
|---|---|
| `estate.nshkrdotcom` | Repositories under the `nshkrdotcom` owner |
| `estate.north_shore_ai` | Repositories under the `North-Shore-AI` owner |
| `platform.nshkr` | The NSHKR platform |
| `seam.dependency_sources` | Repositories carrying an installed dependency-source helper |
| `seam.zero_dependency` | Helper-carrying repositories that declare no cross-repository dependency at all |
| `release.train.core` | Repositories providing packages that publish as one train |
| `family.execution_plane` | The execution and ground plane |
| `family.provider_sdk` | Model and CLI provider SDKs |
| `family.service_connector` | Connectors to third-party services |
| `family.self_hosted_inference` | Self-hosted inference and model serving |
| `family.observability` | Tracing, replay, and introspection |
| `family.workspace_tooling` | Tooling that operates on the portfolio rather than shipping in it |
| `family.jido` | The Jido agent platform |
| `family.crucible` | The Crucible evaluation stack |

Every repository carries at least one group, and no group carries every
repository.

## What a view cannot change

Scope only. A view cannot change a dependency edge, a source, a version
constraint, a release version, or package contents.

## Schema versions

`portfolio_registry.view/v2` is current. `mix_workspace_ops.view/v1` still
loads; its `tags_any` and `tags_all` match a v1 document's project tags and a v2
document's repository groups. Acceptance of v1 is scheduled to be removed after
**2027-02-23**.
