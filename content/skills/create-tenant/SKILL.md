---
name: create-tenant
description: Standardize scaffolding a new co-located tenant workspace (canon SA-022/SA-023) — a container whose ROOT is the orchestrator (rasa.tenant.core brain installed at .claude/, optionally with a coordination module) and which holds codebase member repos as co-located siblings. Interview the author for the namespace, the brain/coordination modules, and the member roster, run bin/build-tenant, then walk installing the brain at the root, bringing in each member + its domain, registering members, and shipping. Supersedes the pre-fold /build-tenant. Triggered by "/create-tenant", "create a tenant", "new tenant", "scaffold a tenant", "set up a company/firm tenant", "build a co-located workspace".
---

# /create-tenant — Scaffold a new co-located tenant workspace

Stand up a **co-located tenant** (canon SA-022, reconciled to SA-023): a container
directory whose **root IS the orchestrator** — `rasa.tenant.core` (the brain)
installs at the tenant root's `.claude/`, optionally with a coordination module
(`rasa.module.firm`/`company`/…) on top — and which holds its **codebase member
repos** as co-located siblings (`<ns>-<name>/`), each its own git repo.

This **supersedes the pre-fold `/build-tenant`.** That skill — and the copy of
`bin/build-tenant` still shipped in `module-workspace` (`rasa.module.workspace`,
a live element whose SA-022-era build-tenant is **pending removal** now that this
tooling lives in `tenant-core`) — is SA-022-era: it scaffolds a separate
`<ns>-cto` *orchestrator member* and defaults `rasa.orchestrator.cto`.
SA-023 removed all of that — the tenant root is the orchestrator, there is no
orchestrator member, and the `orchestrator` kind is gone (`orchestrator.cto` →
`module.cto`, `.firm`/`.company` → `module.*`). Use `/create-tenant`.

The tenant sibling of `/create-domain` (a knowledge Element) and `/create-module`
(a mountable capability). A tenant is neither — it is the **deployment container**
those Elements are installed *into*.

## What makes a tenant distinct (read this first)

- **The root IS the orchestrator (SA-023 FD-002).** One tenant = one
  composition-root/brain. `rasa.tenant.core` installs at the tenant root
  (`.claude/`); coordination modules (`module.firm`/`company`/`cto`) layer on top.
  **No separate orchestrator member** — that was the pre-fold model.
- **Not a fork.** Unlike a domain/module (forked from `domain-core`), a tenant is
  a **thin config container** written fresh by `bin/build-tenant` — `rasa.json`
  (`kind: tenant`, `tenant.layout: co-located`, `tenant.members[]`) + `CLAUDE.md`
  + `README.md` + `.gitignore` + `VERSION`. No `content/`, no `bin/init` of its own.
- **Members are their own repos.** Each `<ns>-<name>/` codebase member is a
  separate git repo with its own remote; the tenant **gitignores** them
  (git-per-repo is the sync — CW-001). The tenant versions only its thin layer.
- **Elements install INTO the right place** (CW-006): the **brain + coordination
  modules install at the tenant root**; a **domain installs INTO a codebase
  member** (`rasa.domain.devops` → `<ns>-devops/`).
- **`tenant.members[]` ≠ `requires.elements[]`.** The first lists *which repos
  live here* (the physical roster); the second lists *which Elements are
  activated* (`tenant.core` + coordination modules + member domains).

## Parameters (the form)

Collect these up front — one prompt each, sensible defaults, **confirm the full
set before writing**. Only `namespace` + `--dir` are required; the rest refine
the roster and are applied by editing `rasa.json` / walking installs after the
scaffold.

| Parameter | Required | Default | What it is / where it lands |
|---|---|---|---|
| `namespace` | yes | — | Short slug `^[a-z][a-z0-9-]*$` (e.g. `vsi`, `acme`). Supplies the `<ns>-` member prefix + `rasa.namespace`. → positional arg. |
| `dir` | yes | — | Target directory for the tenant container. → `--dir`. |
| `name` | no | `rasa.tenant.<ns>` | `rasa.json#name`. **First-party** → `rasa.tenant.<ns>`; **customer** → `<publisher>.tenant.<id>` (§3). → `--name`. |
| `brain` | no | — | Coordination module(s) installed at the root **on top of** `rasa.tenant.core` — `rasa.module.company` (across-engagements), `rasa.module.firm` (law-firm), etc. Repeatable. → `--brain`. |
| `members` | no | `[]` | Codebase members, each `<name>\|role[\|git_remote[\|installed_element]]`. A bare `<name>` is auto-prefixed to `<ns>-<name>` (CW-002). Repeatable → `--member`. A member with a `git_remote` is cloned; without one it is scaffolded. |
| `purpose` | no | `company-engineering-substrate` | `tenant.purpose`. → `--purpose`. |
| `owner` | no | `$USER` | `tenant.owner`. → `--owner`. |
| `contract_version` | no | `1.3.0` | The fleet convention until the v1.4.0 lock — don't override without reason. → `--contract-version`. |
| `visibility` | no | `private` | `public` \| `private` for the tenant repo + each scaffolded member at first push. |

## Interview

Gather the parameters one prompt at a time, sensible defaults, and **confirm the
collected set before writing**:

1. **Namespace + target dir?** (validate the slug; show the derived
   `rasa.tenant.<ns>` name and the `<ns>-*` member prefix. Confirm `<dir>` is
   empty — `bin/build-tenant` refuses a non-empty dir.)
2. **First-party or customer?** — `rasa.tenant.<ns>` vs `<publisher>.tenant.<id>`
   (decides `--name` and which GitHub org the repos land on).
3. **Coordination brain?** — `rasa.tenant.core` is always the base brain; add a
   coordination module if the tenant coordinates work (`module.company` /
   `module.firm` / …), or none for a plain container.
4. **Members?** — the codebase repos: name, role, and either an existing
   `git_remote` (adopt/clone) or none (scaffold a new one). Which domain (if any)
   installs into each.
5. **Purpose / owner / visibility?** (defaults offered).

Echo the full set back and get a go before running the scaffold.

## Prerequisites

- **Run from inside the substrate workspace**, with `elements/tenant-core/`
  present (this skill's Element — it carries `bin/build-tenant` + `bin/init`).
- The **brain + coordination modules + member domains** must be reachable to
  install (materialized under `elements/`, or pull them first). `bin/build-tenant`
  only *declares* them in `requires.elements[]`; the installs are walked steps.

## Implementation

Thin wrapper around [`bin/build-tenant`](../../../bin/build-tenant) — run it from
the workspace root at its real path, `elements/tenant-core/bin/build-tenant`. The
script writes the thin tenant config layer + declares the codebase member roster;
it does **not** clone codebases, create GitHub repos, or install Elements (those
are the walked follow-ups).

```
elements/tenant-core/bin/build-tenant <namespace> --dir <path> \
  [--name <name>] [--brain rasa.module.<x>]... \
  [--member "<ns>-<name>|<role>|<git_remote>|<installed_element>"]...
```

(`--dry-run` prints the `rasa.json` it would write and changes nothing — run it
first; `--contract-version` defaults to `1.3.0`.)

It produces (no orchestrator member — the root is the orchestrator):

- `<dir>/rasa.json` — `kind: tenant`, `tenant.layout: "co-located"`,
  `tenant.members[]` (codebase members only), `requires.elements[]`
  (`rasa.tenant.core` + `--brain` modules + member domains),
  `contract_version: 1.3.0`.
- `<dir>/CLAUDE.md` + `README.md` — thin orientation ("the tenant root IS the
  orchestrator").
- `<dir>/.gitignore` — ignores every member repo (`<ns>-*/`).
- `<dir>/VERSION` — `0.1.0`. `git init -b main`, no commit, no remote.

## Process

Walk the author through the follow-ups below, **confirming before any
side-effectful step** (git, `gh`, install, register), then present the completion
card. `bin/build-tenant` runs from the workspace root; installs run the target
Element's `bin/init`.

1. **Preview + scaffold.** `elements/tenant-core/bin/build-tenant <ns> --dir
   <path> … --dry-run`, confirm the `rasa.json`, then drop `--dry-run`.
2. **Install the brain at the ROOT** (the root is the orchestrator):
   `elements/tenant-core/bin/init <dir>` (installs `rasa.tenant.core` into the
   tenant root's `.claude/`). Then each coordination module:
   `<path-to-module>/bin/init <dir>`.
3. **Bring in each codebase member** as a co-located sibling:
   - **adopt** an existing repo → `git clone <git_remote> <dir>/<ns>-<name>`;
   - **scaffold** a new one → `gh repo create <org>/<ns>-<name>` + `git init` at
     `<dir>/<ns>-<name>/` (a `/new-repo` convenience skill is a planned CW-005
     addition; use the manual steps until it ships).
4. **Install each member's domain INTO the member** (CW-006):
   `<path-to-rasa.domain.X>/bin/init <dir>/<ns>-<name>` (e.g. `rasa.domain.devops`
   → `<ns>-devops/`). The brain/modules stay at the root; domains go in members.
5. **Register the members** by hand-editing `rasa.json#tenant.members` to match
   what's on disk (a `/register` convenience skill is a planned CW-005 addition).
   Confirm `tenant.members[]` reflects every `<ns>-*` member sibling.
6. **Ship** — gated on explicit confirmation, and **don't push from the sandbox**
   (hand pushes to the user / the prepared script):
   - The **tenant** repo: `gh repo create <org>/<tenant-repo> <--public|--private>
     --source=<dir> --remote=origin --push` — `<org>` is **RasaOS** for a
     first-party tenant, the **customer's org** for a `<publisher>.tenant.<id>`
     tenant.
   - **Each scaffolded member** (adopted members already have remotes): create its
     repo + push, per its own org.
7. **Register in the workspace records** (first-party tenants): add the tenant row
   to `elements/REGISTRY.md`, an entry to `elements/CHANGELOG.md` (track #2), and a
   line to `canon/AUDIT.md`. (Customer tenants live in the customer's org, not the
   RasaOS roster — skip the RasaOS REGISTRY for those.)

## Completion card

After the run, present:

- **Scaffolded:** `<dir>/` — `rasa.tenant.<ns>` v0.1.0, `layout: co-located`,
  brain `rasa.tenant.core`(+ modules), members `<collected>`.
- **Follow-ups, each confirmed before acting:**
  1. Brain installed at the root (`tenant.core` + coordination modules).
  2. Members cloned/scaffolded as `<ns>-*` siblings; each member's domain
     installed INTO it.
  3. `tenant.members[]` registered + reconciled with disk.
  4. Tenant repo + each scaffolded member created & pushed (gated; correct org;
     outside the sandbox).
  5. Registered in the workspace records (first-party only).

## Canon

- **SA-018** — `tenant` as an Element kind (the deployment-level composition root).
- **SA-022 + SA-023** — the **co-located workspace** flavor (CW-001..CW-006:
  member repos as siblings, `<ns>-<name>` naming, `tenant.members[]`, orchestrators
  create+register members, Elements install INTO members) **reconciled to the
  fold**: the tenant root IS the orchestrator (`tenant.core` at `.claude/`), no
  separate orchestrator member. `ELEMENT_CONTRACT` §8b.
- **SA-019** — the other tenant flavors (holding-folder + canon-author full-clone);
  see *When NOT to use this skill*.
- **Seven kinds** (SA-023): `domain | module | core | kernel | frontend | recipe |
  tenant`; the `orchestrator` kind is gone.

## When NOT to use this skill

- **Pulling a managed Element into a customer tenant with smart-merge sync** →
  that's the **holding-folder** flavor (SA-019: visible `elements/<name>/` + hidden
  `.rasa/holding/<name>/`), not a co-located workspace.
- **The canon-author / first-party full-clone tenant** (`rasa.tenant.rasaos`, which
  holds `elements/` as full clones) → that's the SA-019 canon-author flavor,
  materialized differently.
- **Adding one repo to an existing tenant** → scaffold/adopt the member repo and
  add it to `tenant.members[]` directly (the planned `/new-repo` + `/register`
  CW-005 skills), not `/create-tenant`.
- **A knowledge Element or a capability** → `/create-domain` or `/create-module`.

## What NOT to do

- Don't scaffold an `<ns>-cto` orchestrator member or default `orchestrator.*` —
  that is the retired pre-SA-023 model. The **root** is the orchestrator.
- Don't run `gh repo create`/commit/push without explicit confirmation — and don't
  push from the sandbox (hand it to the user). Put a first-party tenant on
  **RasaOS**, a customer tenant on the **customer's** org.
- Don't install domains at the tenant root — domains install **INTO members**
  (CW-006); only the brain + coordination modules install at the root.
- Don't commit member repos into the tenant — they are gitignored, git-per-repo.
- Don't bump `contract_version` past the fleet convention (`1.3.0`) until the
  coordinated v1.4.0 lock.
- Don't author `MANIFEST.json` or any forbidden legacy vocabulary (§8).

## Done when

`<dir>/` exists as a co-located tenant (`kind: tenant`, `layout: co-located`) with
`rasa.tenant.core` (+ any coordination modules) installed at the **root**, each
codebase member present as a `<ns>-*` sibling with its domain installed INTO it,
`tenant.members[]` reconciled with disk, the tenant repo + scaffolded members
created and pushed to the correct org (or the exact steps handed to the user), and
— for a first-party tenant — registered across `elements/REGISTRY.md`,
`elements/CHANGELOG.md` (track #2), and `canon/AUDIT.md`.
