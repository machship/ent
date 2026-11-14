---
lastReviewed: 2026-11-14
---
# ent (MachShip fork)

`machship/ent` is MachShip's fork of [ent](https://entgo.io), the Go entity framework
(`github.com/ent/ent`). We forked it to add a **SQL Server dialect**, so Go services can
generate typed ent clients over MachShip's existing SQL Server tables. It is a library: nothing
is deployed. Owned by uplift; GitHub project-type Proof of Concept.

## What MachShip changed

The changes are on branch **`sqlserver`**, not on the default branch `master`.

| Commit | Date | Change |
|---|---|---|
| `bda390d16` | 2025-10-17 | Add SQL Server dialect: `dialect.SQLServer = "sqlserver"`, SQL Server builder support in `dialect/sql/builder.go`, and `dialect/sql/schema/sqlserver.go` for migrations |
| `a4f8d9545` | 2025-10-27 | Build the `OUTPUT` clause (returning inserted/updated rows) in one place (PR #2) |
| `0a2cc7d97` | 2025-12-15 | Compare `bit` columns explicitly in `WHERE` clauses (PR #5) |
| `62acc342f` | 2026-01-29 | Support SQL Server table hints (for example `WITH (NOLOCK)`) |
| `59cba6396` | 2026-10-27 | Merge upstream `ent/ent` master into `sqlserver` (PR #9) |

In total about 800 lines across `dialect/dialect.go`, `dialect/sql/builder.go` (+ tests),
`dialect/sql/schema/atlas.go` and `dialect/sql/schema/sqlserver.go`.

`master` is upstream `ent/ent` master plus MachShip's Dependabot fixes for the Go modules and
the docs website (PR #10, Nov 2026 Docusaurus cleanup). Those fixes are not on `sqlserver`.
Other branches: `sync/upstream-2026-10` and `fix/sqlserver-output` are merged work branches.

## Who imports it

Consumers keep `entgo.io/ent` in their imports and swap the module with a `replace` directive:

```
replace entgo.io/ent => github.com/machship/ent v0.0.0-<date>-<sqlserver commit>
```

| Repo | Pinned commit (on `sqlserver`) | Use |
|---|---|---|
| machship-v4 (`github.com/machship/tms-poc`) | `59cba639` (2026-10-27) | Maps existing MachShip SQL Server tables (`carrier`, `companycarrier`) through ent |
| data-gateway | `ef4724cde5ee` (2025-12-15) | Typed access to MachShip SQL Server data |
| machship-go (archived) | `62acc342f424` (2026-01-29) | |

golden-repo-poc uses upstream `entgo.io/ent` (Postgres), not the fork.

## Context

See [diagrams/context.md](diagrams/context.md). On the map this component has only package
lines: importers list `github.com/machship/ent` under `packages`.

## Publishing

No release workflow and no tags of our own. Consumers fetch a pseudo-version from the
`sqlserver` branch through the Go toolchain with `GOPRIVATE=github.com/machship`. The upstream
CI workflows (`ci.yml`, `atlas-ci-public.yaml`) are still in the repo.

## Running in k8s

Nothing. No image, no workload, no secrets.

## Key decisions

- Fork rather than wait for upstream: ent has no SQL Server dialect.
- Keep the import path `entgo.io/ent` in consumers and switch with `replace`, so generated
  code and upstream docs still apply.

## Lifecycle notes

- To pick up upstream changes, merge `ent/ent` master into `sqlserver` (last done 2026-10-27)
  and bump the pseudo-version in each consumer.
- TODO(confirm): should `sqlserver` become the default branch, or be offered upstream?
