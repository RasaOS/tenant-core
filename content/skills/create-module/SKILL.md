---
name: create-module
description: Standardize creating a new module-kind Element — a focused, mountable, opt-in capability that extends a tenant or a domain via a project-owned adapter seam — by interviewing the author for name, description, parent_kind (default ["domain","tenant"]), and identity, running bin/new-element module <name>, and walking the full lifecycle (fill rasa.json + register content in element.files[], author content/, git add + check-manifest + check-shape, first publish or bin/ship-element on later bumps, register via bin/refresh-registry). Triggered by "/create-module", "create a module", "new module element", "scaffold a module", "make a mountable capability".
---

# /create-module — Scaffold a new module-kind Element

Stand up a new **module** Element: a focused, opt-in capability that **mounts
into** a parent (a tenant or a domain) rather than standing alone. Modules are
the successor to the legacy `skill` kind — small, portable, parent-mountable.
The 15 shipped modules (`rasa.module.tasks`, `.releases`, `.pipelines`, …) are
the reference roster.

This is a wrapper: it interviews the author, runs the real
[`bin/new-element`](../../../bin/new-element) `module` path, then walks the
standardized lifecycle end-to-end (fill → validate → publish → register),
confirming before every side-effectful step. The steps below are ordered and
parameterized on purpose — a UI can render the **Parameters** block as a form
and drive the **Process** as a checklist.

Sibling skills, so you pick the right one: a **domain** is authored with
`/create-domain` (a broad knowledge/toolkit surface **installed** into a
consumer's `.claude/`); tenant scaffolding is a separate skill (`/create-tenant` —
it supersedes the pre-SA-023 `/build-tenant`). `/create-module` is
for the narrow case — **one capability, mounted into a parent, enabled by hand.**

## What makes a module distinct from a domain (read this first)

A module is not a small domain. Four traits define the kind — hold them in mind
while authoring `content/`:

- **`requires.parent_kind`** — the module-kind-only field (Contract v1.4.0 §6).
  An array that must be a **subset of `["domain", "tenant"]`** and names which
  parent kinds the module may mount into. Omitted → both. Per **SA-023** the
  `orchestrator` kind was folded into `tenant`, so modules now mount into a
  **tenant** or a **domain** (the old `["domain","orchestrator"]` default is
  gone). A domain has no `parent_kind` — it *is* a top-level surface.
- **Mountable, opt-in capability** — a module is *pulled/mounted* into a parent
  and is registered-but-not-auto-installed (install policy `opt-in`, §7); a
  human enables it. A domain, by contrast, is **installed** (Pattern 1) —
  `bin/init` copies its `content/` into a consumer's `.claude/` and the consumer
  owns the result.
- **Adapter-seam pattern** — a module delegates its **per-project concern to a
  project-owned adapter seam**: the module ships the generic capability logic and
  calls into a concrete implementation that the consuming parent/project
  supplies. Keep the module subject-neutral; push anything project-specific out
  to the seam. (Honesty note: "adapter seam" is the workspace roster's design
  language for the module pattern, **not** a formally defined field in
  ELEMENT_CONTRACT v1.4.0 — there is no `rasa.json` key for it. It's how you
  *shape* `content/`, not something the manifest declares.)
- **Focused scope** — one capability, not a corpus. If the thing you're building
  is a broad knowledge/toolkit surface, it's a **domain**, not a module — stop
  and use `/create-domain` instead.

## Parameters (the form)

Collect these up front — one prompt each, sensible defaults, **confirm the full
set before writing anything**. Only `name` + `description` are `bin/new-element`
arguments; the rest are applied by editing `rasa.json` / authoring `content/`
after the scaffold.

| Parameter | Required | Default | What it is / where it lands |
|---|---|---|---|
| `name` | yes | — | Short identifier, validated `^[a-z][a-z0-9-]*$` (no dots). Becomes canonical `rasa.module.<name>`, folder `module-<name>`, display `RasaOS Module · <Name>` (Contract §3). → positional arg to `bin/new-element`. |
| `description` | yes | — | One-line `rasa.json#description`. → `--description` flag. (If omitted the scaffolder writes a `(TODO: …)` placeholder — a fallback to replace, **not** a default; always collect a real value.) |
| `parent_kind` | no | `["domain","tenant"]` | The mount targets — a **subset of `["domain","tenant"]`**. The scaffolder seeds the default; only collect this to **restrict** (e.g. `["domain"]` = domain-only). → edit `rasa.json#requires.parent_kind`. |
| `capabilities` | no | `[]` | Optional array of dotted capability strings naming the focused capability, e.g. `tasks.lifecycle` (§6 — no module-specific rule beyond the dotted-name convention). → edit `rasa.json#capabilities`. |
| `identity` | no | `{one_liner, role, subject, goal}` (suggested: role=`module`, subject=`<name>`) | The fork inherits `domain-core`'s **template** identity verbatim — replace it wholesale. Renders to `.claude/rasa-identity.md` at install (SA-025). → edit `rasa.json#rasa.identity`. |
| `adapter_seam_concern` | no | — | Design input, **not** a `rasa.json` field: the per-project concern the module delegates to the parent-owned seam. Shapes how you author `content/`. |
| `visibility` | no | `private` | `public` \| `private` — governs `gh repo create` at first publish. The shipped module roster is **public**; confirm per module. |

Modules stay on the `domain-core` baseline shape at v0.1.0 — there is no separate
module template; `bin/new-element` forks `domain-core` for every scaffoldable
kind and overrides `kind: "module"`.

## Interview

Gather the parameters one prompt each, sensible defaults, and **confirm the full
set before writing**. **Verify `elements/module-<name>/` does not already exist**
(`bin/new-element` refuses to overwrite; a taken name should fail here as a form
error, not later as a scaffold crash — `--dry-run` prints the target folder).

## Prerequisites

- **Run from inside the materialized tenant** — `bin/new-element` walks up for the
  `kind: tenant` `rasa.json`; if none is found it falls back to your cwd and
  aborts `template folder … not found`. Pass `--workspace <tenant-root>` +
  `--dry-run` if unsure.
- **`elements/domain-core` must be present** (every scaffoldable kind forks it) —
  else the scaffold aborts. Clone `RasaOS/domain-core` into `elements/` first if
  the tenant was only partially materialized.

## Implementation

Thin wrapper around [`bin/new-element`](../../../bin/new-element) — run it from
the workspace root at its real path, `elements/tenant-core/bin/new-element`. The
script copies the `domain-core` template tree into `elements/module-<name>/`,
rewrites `rasa.json` (`name`, `version: 0.1.0`, `description`, `aliases`,
`kind: "module"`, `rasa.role: "L1-mountable-capability"`), **seeds
`requires.parent_kind: ["domain","tenant"]`** (module kind only), resets
`VERSION`/`CHANGELOG.md`/`README.md`, and runs `git init -b main`. It does **not**
commit, wire a remote, create a GitHub repo, or register anything. `contract_version`
and `rasa.identity` are inherited from the template unchanged.

```
elements/tenant-core/bin/new-element module <name> --description "<one-liner>"
```

Useful flags: `--dry-run` (print the planned template/folder/name/description and
make no changes — run this first, it also prints the target folder so a
name-collision surfaces before you write), `--workspace <path>` (override the
tenant-root walk-up).

## Process

Interview first, then work top-to-bottom. `bin/new-element` /
`bin/ship-element` / `bin/refresh-registry` run from the **workspace root** at
`elements/tenant-core/bin/…`; `bin/check-manifest` / `bin/check-shape` run from
**inside the fork**. Every step that writes remote state (commit, `gh repo
create`, push, register) is **gated on explicit confirmation** — and per the
workspace rule, **don't push from the sandbox**: hand the push to the prepared
script or the user.

1. **Interview + confirm** (see *Interview* above) — collect the Parameters, echo
   them back, and verify `elements/module-<name>/` does not already exist.
2. **Preview**: `elements/tenant-core/bin/new-element module <name> --description
   "…" --dry-run`. Confirm the folder/name look right. (If the folder already
   exists the tool refuses — pick a different name, or if you meant to extend an
   existing module stop and use `ship-element` to bump it.)
3. **Scaffold**: drop `--dry-run`. Produces `elements/module-<name>/` — a fresh
   `domain-core` fork with `kind: "module"`, `requires.parent_kind` seeded,
   git-inited, **uncommitted, nothing staged, no remote**.
4. **Fill `rasa.json`** in `elements/module-<name>/`:
   - `requires.parent_kind` — leave `["domain","tenant"]`, or narrow to the
     collected subset (domain-only / tenant-only).
   - `capabilities[]` — the dotted capability string(s).
   - Replace the placeholder `description` if one was scaffolded.
   - `rasa.identity` — **replace `domain-core`'s inherited template identity**
     `{one_liner, role: "module", subject: "<name>", goal}`; it renders to
     `.claude/rasa-identity.md` at install (SA-025), so an unedited fork declares
     your module as the domain template.
   - `contract_version` — **leave the inherited `1.3.0`**: it is the current
     locked + published contract (the `element-contract` mirror's latest tag is
     `v1.3.0`; canon **v1.4.0 is IN PROGRESS**/unpublished). Author to the current
     rules (canon v1.4.0, post-SA-023) but declare the last-published version; the
     fleet migrates to `1.4.0` together at the v1.4.0 lock. The README rendering
     `Element Contract v1.3.0` is correct — **don't bump it.**
   - `permissions[]` — the fork inherits `["fs:write","shell:exec"]`; extend it
     (`git:write`, `network:fetch`, `hooks:install`, `identity:bind`) to match
     what the module's `content/` actually does.
   - `requires{}` — if the module depends on `rasa.core` or other Elements,
     declare them in `requires.elements[]` (distinct from `requires.parent_kind`).
   - `element.files[]` — see step 5; register authored content here.
5. **Author `content/`** for the one capability, shaped around the **adapter
   seam** — generic module logic here, the project-specific concern delegated to
   a seam the parent supplies. Then:
   - **Register new content in `element.files[]`:** every git-tracked file under
     `content/` (and `seed/`) must be covered by an exact `from` or an
     **ancestor-directory** `from` (a directory entry covers its whole subtree).
     Flip the dirs you ship from `opt-in` → `directory-mirror`. Worked entry:
     `{ "from": "content/skills/", "to": ".claude/skills/", "policy": "directory-mirror" }`.
   - **Delete the inherited `content/SHAPE.md`** and write a `content/README.md`
     for this module. **Also remove `content/SHAPE.md` from `element.files[]`**
     (repoint the `content/README.md` entry at your fork's README) — otherwise
     `check-manifest` reports it **DANGLING** and the gate goes red.
   - Rewrite the Element's own root `README.md` + `CLAUDE.md` for this module (the
     fork inherits the template's; note the root `CLAUDE.md` is distinct from
     `seed/CLAUDE.md.template`, the consumer-facing project file). The fork also
     inherits the generic `/sync` + `/promote` + `/whoami` skills, which run
     against the `kit/<element>/` clone `bin/init` drops at install (SA-024);
     they only reach consumers if `content/skills/` is `directory-mirror` and
     `source.repo` points at the real upstream.
6. **Confirm the contract version + review the LICENSE.** Leave `contract_version`
   at the inherited `1.3.0` (step 4 — the current published contract; don't bump
   to 1.4.0 until the fleet lock). Review the inherited Apache-2.0 `LICENSE` (the
   public template's) — for a private/proprietary module, replace or drop it to
   match visibility before the repo is created. Fill the scaffolded
   `CHANGELOG.md` v0.1.0 entry (real date, remove the `(TODO: date)` marker + the
   "TODO before first ship" checklist, describe what ships). *(Optional, §11a:
   drop a local `ELEMENT_CONTRACT.md` matching `contract_version`:
   `curl -O https://raw.githubusercontent.com/RasaOS/element-contract/v1.3.0/ELEMENT_CONTRACT.md`.)*
7. **Stage, then validate.** `check-manifest` inventories only **git-tracked**
   files, so **`git add -A` first** — before staging it reports "0 tracked files"
   and passes *vacuously*. Then from inside `elements/module-<name>/`:
   - **`bin/check-manifest`** must exit **0** — every tracked file under
     `content/`/`seed/` is covered in `element.files`/`seed.files`, no `from`
     dangles. Failure classes: **UNREGISTERED** (add the file/dir with a policy)
     and **DANGLING** (fix the `from` path or drop the entry). Fix until green
     (Contract §9).
   - **`bin/check-shape`** — the second, co-equal release gate (STRUCTURE to
     check-manifest's INVENTORY), inherited from the fork. It errors if any
     `content/skills/<name>/SKILL.md` lacks `## Behavior contract`, `## Process`,
     `## What NOT to do`, `## Done when`, or has a stub description, and validates
     rule filenames/H1s + agent frontmatter (and WARNs — non-fatal — if you add a
     skill without updating the enumerating note on the `content/skills/`
     `element.files[]` entry). It passes clean on a fork with no
     skills/rules/agents. **`ship-element` runs only `check-manifest`, so run
     `check-shape` by hand before tagging.**
   - **§9 basics:** valid JSON, `name` matches the §3 regex, `kind` is exactly
     `module`, `version` mirrors `VERSION`, no forbidden §8 vocabulary.
8. **Publish** — two cases, and they differ:
   - **First release (v0.1.0):** `bin/ship-element` is *not* for this (it bumps
     the version and pushes to an existing remote). Create the repo, wire
     `origin`, and push in one step:
     ```
     git add -A && git commit -m "v0.1.0: initial" && git tag v0.1.0
     gh repo create RasaOS/module-<name> <--public|--private> --source=. --remote=origin --push
     git push --tags
     ```
     Confirm visibility (the shipped module roster is **public**). If
     `RasaOS/module-<name>` already exists, skip `gh repo create` — just
     `git remote add origin …` (if unset) and push. **Gate the `gh repo create` +
     push on explicit confirmation; push via the prepared script / the user, not
     from the sandbox.**
   - **Every later bump:** `elements/tenant-core/bin/ship-element module-<name>
     --bump <patch|minor|major> -m "…"`. It preflights (clean tree — else
     commit/stash; on `main` — else `git checkout main`; `check-manifest` passes;
     `origin` must exist), bumps `rasa.json#version` + `VERSION`, inserts a dated
     `CHANGELOG.md` skeleton and pauses for you to fill it, commits + tags
     `vX.Y.Z`, pushes, and prints paste-ready `elements/REGISTRY.md` /
     `elements/CHANGELOG.md` / `canon/AUDIT.md` snippets.
9. **Register** — a new element ship is a **track #2** event.
   `elements/tenant-core/bin/refresh-registry --check` (exits 1 on drift), then
   `--write` to regenerate the `<!-- BEGIN AUTO: live-elements -->` region of
   `elements/REGISTRY.md` from live disk. Add the entry to `elements/CHANGELOG.md`
   (track #2), append a line to `canon/AUDIT.md` (the per-action ledger), and add
   the new sibling row to the workspace `CLAUDE.md` Zone A table. The workspace
   `CHANGELOG.md` is **track #1** (canon / orchestration work) — a lone module
   scaffold doesn't touch it. (`ship-element` prints these snippets for you on
   bumps.)

## Completion card

After the run, present:

- **Scaffolded:** `elements/module-<name>/` — `rasa.module.<name>` v0.1.0,
  `kind: module`, `requires.parent_kind: <collected>`, `capabilities: <collected>`.
- **Follow-ups, each confirmed before acting:**
  1. `rasa.json` filled (`parent_kind`, `capabilities`, real `description`,
     `identity`, `contract_version`, `permissions`) + authored content registered
     in `element.files[]`.
  2. `content/` authored around the adapter seam; `SHAPE.md` deleted **and
     deregistered** → `content/README.md`; root `README.md` + `CLAUDE.md`
     rewritten; `LICENSE` reviewed.
  3. `contract_version` left at the inherited `1.3.0` (fleet convention until the
     v1.4.0 lock); `CHANGELOG.md` v0.1.0 entry filled.
  4. `git add -A`, then `bin/check-manifest` **and** `bin/check-shape` green.
  5. First publish (`git commit`/`tag v0.1.0`/`gh repo create --source=. --push`)
     **or** `bin/ship-element` on a later bump — pushes gated + run outside the
     sandbox.
  6. Registered: `bin/refresh-registry --write` + `elements/CHANGELOG.md`
     (track #2) + `canon/AUDIT.md` line + workspace `CLAUDE.md` Zone A table row.

## Canon

- **Element Contract v1.4.0** — §2 (`kind` must be exactly `module`), §3 (naming
  `rasa.module.<name>`, folder `module-<name>`), §4 required files (`content/`
  REQUIRED for `module`; plus `rasa.json`, `VERSION`, `README.md`, `CLAUDE.md`,
  `CHANGELOG.md`, `.gitignore` — all inherited from the fork), §5 (five required
  `rasa.json` fields: `$schema`, `contract_version`, `name`, `version`, `kind`),
  §6 (`requires.parent_kind` = the module-only field; `capabilities`,
  `permissions`, `element.files`), §7 (`opt-in` install policy), §9 (validation),
  §10 (versioning).
- **SA-023** — the seven kinds (`domain`, `module`, `core`, `kernel`,
  `frontend`, `recipe`, `tenant`); the `orchestrator` kind **folded into
  `tenant`**; modules mount into a **tenant or a domain** (`parent_kind ⊆
  ["domain","tenant"]`). Modules replace the legacy `skill` kind.
- **SA-024** — `bin/init` drops a pinned `kit/<element>/` clone powering `/sync`
  + `/promote`. **SA-025** — `rasa.identity` renders to `.claude/rasa-identity.md`
  at install.

## When NOT to use this skill

- **It's a broad knowledge/toolkit surface, not one capability** → author a
  **domain** with `/create-domain`, not a module.
- **You're scaffolding the container, not a capability** → `/create-tenant`
  (supersedes `/build-tenant`).
- **You're adding a skill/rule to an existing Element** → that's authoring
  `content/` inside that Element, not creating a new module.
- **`core` / `kernel` / `tenant`** → out of scope for `/create-module`. Note
  `core` *is* scaffoldable (`bin/new-element core` — a singleton, name forced to
  `rasa.core`); only `tenant` and `kernel` have separate, non-`new-element`
  scaffolds.

## What NOT to do

- Don't run `gh repo create`, commit, tag, or push without explicit
  confirmation — and don't push from the sandbox (hand it to the user).
- Don't run `bin/check-manifest` before `git add` — it passes vacuously on an
  unstaged tree ("0 tracked files").
- Don't ship on a red `bin/check-manifest` or `bin/check-shape` — exit 0 on both
  is the gate.
- Don't delete `content/SHAPE.md` without also removing its `element.files[]`
  entry (DANGLING).
- Don't leave the template's `rasa.identity`, `LICENSE`, or `content/SHAPE.md`
  unreviewed in the fork.
- Don't invent `bin/new-element` flags — it takes only `<kind> <name>
  [--description] [--dry-run] [--workspace]`.
- Don't use `bin/ship-element` for the initial 0.1.0 ship.
- Don't author `MANIFEST.json` or any forbidden legacy vocabulary (§8).

## Done when

`elements/module-<name>/` exists as a fresh `domain-core` fork with a filled
`rasa.json` (`requires.parent_kind`, `capabilities`, real `description`,
`identity`, `contract_version`, `permissions`, and every authored file covered by
`element.files[]`), `content/` authored around the adapter seam (SHAPE.md removed
*and deregistered*, a fork-specific `content/README.md` in place), `bin/check-manifest`
**and** `bin/check-shape` both exit 0 on a staged tree, the module is shipped at
v0.1.0 with `origin` wired (or the exact first-ship steps handed to the user),
and it's registered across `elements/REGISTRY.md`, `elements/CHANGELOG.md`
(track #2), `canon/AUDIT.md`, and the workspace `CLAUDE.md` Zone A table.
