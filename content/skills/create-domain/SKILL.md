---
name: create-domain
description: Standardize creating a new domain-kind Element — interview the author for the domain name, a one-line description, and a shape pattern (toolkit / structural / hybrid), run the real `bin/new-element domain` scaffold, then walk them through filling rasa.json (capabilities + shape_pattern + identity + permissions), registering authored content in element.files[], populating content/ per the chosen pattern, validating with check-manifest + check-shape, shipping, and registering the sibling in elements/REGISTRY.md. Triggered by "/create-domain", "create a domain", "new domain element", "scaffold a domain", "author a rasa domain", "add a domain to the substrate".
---

# /create-domain — Author a new domain-kind Element

Stand up a new **domain** Element (canon Element Contract v1.4.0 §2 — a portable
knowledge/toolkit foundation for a vertical, `rasa.domain.<name>`) the standardized way:
fork the locked `rasa.domain.core` template, pick a shape pattern up front, fill
the manifest + subject content, validate, ship, and register. The step order is
fixed on purpose — this skill is the surface a UI can later render as a form,
so the same flow runs the same way every time.

The domain sibling of tenant-scaffolding (see *When NOT to use this skill*). For
a mountable capability that extends a parent instead of a standalone vertical,
you want a **module** (`/create-module`), not a domain.

## Parameters

The inputs this skill collects. A UI renders these as a form; the interview
below gathers the same values one prompt at a time. Only **name** and
**description** are `bin/new-element` arguments — the rest are applied by editing
`rasa.json` / authoring `content/` after the scaffold, so don't invent flags for
them.

| Parameter | Required | Shape | Feeds |
|-----------|----------|-------|-------|
| **name** | yes | short lowercase slug, regex `^[a-z][a-z0-9-]*$` (e.g. `health`, `soccer-goalkeeping`) — **no dots** (sub-namespacing is handled post-scaffold, below) | positional `<name>` → dotted `rasa.domain.<name>`, folder `domain-<name>`, display `RasaOS Domain · <Name>` (§3) |
| **description** | yes | one line, no trailing period needed | `--description "..."` → `rasa.json#description` + the README title line. (The scaffolder writes a `(TODO: …)` placeholder if omitted — that is a fallback to replace, **not** a default; always collect a real value.) |
| **shape_pattern** | yes | enum: `toolkit` \| `structural` \| `hybrid` | hand-written to `rasa.json#rasa.shape_pattern`; decides the `content/` layout + the `capabilities[]` shape (SHAPE.md) |
| **capabilities** | no | list of dotted strings; may be `[]` | hand-written to `rasa.json#capabilities` — `<domain>.<verb-or-noun>` for toolkit, `<domain>.<concept>` for structural |
| **identity** | no | `{one_liner, role, subject, goal}` (suggested form values, *not* scaffold-filled: role=`domain`, subject=`<name>`) | hand-written to `rasa.json#rasa.identity` — the fork inherits `domain-core`'s template identity verbatim, so replace it wholesale (step 2); renders to `.claude/rasa-identity.md` at install (SA-025) |
| **sub_namespace** | no | dotted `rasa.domain.<area>.<specialty>` (e.g. `soccer.goalkeeping`) | pass the **dashed** slug (`soccer-goalkeeping`) to the scaffold, then hand-edit `rasa.json#name` to the dotted form + add the dashed form to `aliases[]` (step 2) |
| **visibility** | no | enum: `private` \| `public` (default `private`) | the `gh repo create --private`/`--public` flag at first ship |

## Interview

Gather the parameters one prompt at a time, sensible defaults, and **confirm the
collected values before writing**:

1. **Domain name?** (validate the slug; show the derived `rasa.domain.<name>` /
   `domain-<name>` / `RasaOS Domain · <Name>` triple. **Confirm
   `elements/domain-<name>/` does not already exist** — `bin/new-element` hard-
   refuses to overwrite, so a taken name should fail here as a form error, not
   later as a scaffold crash. `--dry-run` prints the target folder to check.)
2. **One-line description?**
3. **Shape pattern?** — `toolkit` (skills/rules/agents, like `rasa.domain.code`),
   `structural` (organization subdirs like firm/ drives/ members/, like
   `rasa.domain.legal`), or `hybrid` (both; no canon example yet). Explain the
   trade before they pick.
4. **Capabilities?** (optional — dotted strings shaped by the pattern; `[]` is
   fine for a first cut).
5. **Sub-namespaced?** (optional — if this is `rasa.domain.<area>.<specialty>`,
   note it now; the name gets the dotted form + a flat alias in step 2).
6. **Identity + visibility?** (optional; suggested role=`domain`, subject=`<name>`,
   visibility=`private`).

Then echo the full set back and get a go before running the scaffold.

## Prerequisites

- **Run from inside the materialized tenant.** `bin/new-element` walks up for the
  `kind: tenant` `rasa.json`; if it finds none it silently falls back to your cwd
  and aborts with `template folder … not found`. If unsure, pass
  `--workspace <tenant-root>` and `--dry-run` first to confirm the *New folder*
  path.
- **`elements/domain-core` must be present** — every scaffoldable kind forks it.
  On a partially-materialized tenant, `git clone https://github.com/RasaOS/domain-core`
  into `elements/` first, or the scaffold aborts with `template folder … not found`.

## Implementation

Thin wrapper around [`bin/new-element`](../../../bin/new-element) (run it from the
workspace root at its real path, `elements/tenant-core/bin/new-element`). The
script forks `elements/domain-core`, rewrites the manifest for the new identity,
and inits a fresh git repo — it does **not** create GitHub repos, commit, wire a
remote, or fill capabilities/content (those are the walked follow-ups).

```
elements/tenant-core/bin/new-element domain <name> --description "<one-line description>"
```

(`--dry-run` prints the plan and writes nothing — run it first; `--workspace <path>`
overrides the root, which otherwise resolves by walking up for the tenant `rasa.json`.)

It produces:

- `elements/domain-<name>/` — a fresh recursive copy of the `domain-core` tree
  (`.git` excluded), with its own new git repo (`git init -b main`, **no remote,
  no commit, nothing staged yet**).
- `rasa.json` rewritten: `name` = `rasa.domain.<name>`, `version` = `0.1.0`,
  your `description`, `aliases` = `["domain-<name>"]`, `kind` = `domain`,
  `tier` = `Pro`, `rasa.role` = `domain-element`, a forked-from `rasa.note`,
  `source.repo` = `https://github.com/RasaOS/domain-<name>`. (`shape_pattern`
  stays `"agnostic"`, `capabilities` stays `[]`, `contract_version` and
  `rasa.identity` are **inherited from the template unchanged** — all yours to fill.)
- `VERSION` reset to `0.1.0`.
- `CHANGELOG.md` overwritten with a fresh `v0.1.0 — INITIAL` entry + a "TODO
  before first ship" checklist (placeholder date — you fill it, step 6).
- `README.md` overwritten with fresh-fork framing.

## Process

Walk the author through the numbered follow-ups below, **confirming before any
side-effectful step** (git, `gh`, register), then present the completion card.
Everything below runs from inside `elements/domain-<name>/` unless noted.

1. **`cd elements/domain-<name>`.**

2. **Fill `rasa.json`.**
   - `rasa.shape_pattern`: replace `"agnostic"` with the chosen pattern.
   - `capabilities[]`: fill with dotted strings shaped per the pattern
     (toolkit → `<domain>.<verb-or-noun>`; structural → `<domain>.<concept>`),
     or leave `[]` for a first cut.
   - `rasa.identity` `{one_liner, role, subject, goal}`: **replace the template's
     identity wholesale** — the fork ships `domain-core`'s verbatim (it describes
     the *template*), and it renders to `.claude/rasa-identity.md` at install
     (SA-025), so an unedited fork misdeclares itself.
   - `contract_version`: **leave the inherited `1.3.0`** — it is the current
     locked + published contract (the `element-contract` mirror's latest tag is
     `v1.3.0`; canon **v1.4.0 is still IN PROGRESS** and unpublished). Author to
     the *current rules* (canon v1.4.0, post-SA-023: seven kinds) but *declare*
     the last-published version; the whole fleet migrates the declaration to
     `1.4.0` together at the v1.4.0 lock (`bin/lock-sequence`), not piecemeal. So
     the scaffolded README rendering `Contract: Element Contract v1.3.0` is
     correct — **don't bump it.**
   - `name` / `aliases[]` (**sub-namespaced domains**): `bin/new-element`'s slug
     regex forbids dots, so for `rasa.domain.<area>.<specialty>` you passed the
     dashed slug — now hand-edit `name` to the dotted canonical form
     (`rasa.domain.soccer.goalkeeping`) and keep the dashed form
     (`domain-soccer-goalkeeping`) in `aliases[]` for flat discoverability (the
     `soccer.goalkeeping` precedent). Review `aliases[]` regardless.
   - `permissions[]`: the fork inherits `["fs:write","shell:exec"]`. Extend it
     (`git:write`, `network:fetch`, `hooks:install`, `identity:bind`) to match
     what the authored `content/` actually does — §9 does not catch under-
     declaration, so it ships silently wrong otherwise.
   - `requires{}`: if this domain depends on `rasa.core` or other Elements,
     declare them in `requires.elements[]` (the `domain-code` precedent).
   - `element.files[]`: see step 3 — you register authored content here.

3. **Populate `content/` per the chosen shape pattern** (SHAPE.md is the guide),
   and **register every file you add in `rasa.json#element.files[]`** — this is
   what makes the validation gate real (step 7):
   - **toolkit** — keep/extend the domain-agnostic starters under
     `content/skills/`, `content/rules/`, `content/agents/`; add `modes/`,
     `build/`, `tests/` as needed. Author skills from
     `content/templates/SKILL.md.template`.
   - **structural** — replace with organization subdirs (e.g. `firm/`,
     `drives/`, `members/`); keep only the Element-mechanics skills that apply.
   - **hybrid** — both; state clearly in `content/README.md` which is which.
   - Add the subject knowledge: **PHILOSOPHY** (stance/tenets), **`framework/`**
     (the structural chapters), and **vocabulary** (the terms).
   - **Register new content:** every git-tracked file under `content/` (and
     `seed/`) must be covered by an `element.files[]` entry — either an exact
     `from` or an **ancestor directory** `from` (a directory entry covers its
     whole subtree, so you needn't list each file). Worked entry:
     ```json
     { "from": "content/framework/", "to": ".claude/<domain>/framework/", "policy": "directory-mirror" }
     ```
     Flip the dirs the fork actually ships from `opt-in` → `directory-mirror` (or
     `file-replace`) so they install; keep `content/templates/` `opt-in`
     (authoring-time only — never auto-install).
   - **Delete `content/SHAPE.md`** and the template's `content/README.md`; write
     the fork's own `content/README.md`. **Also remove the `content/SHAPE.md`
     object from `element.files[]`** (and repoint the `content/README.md` entry
     at your fork's README) — otherwise `check-manifest` flags `content/SHAPE.md`
     as **DANGLING** and the gate goes red.

4. **Author the Element's own `README.md` + root `CLAUDE.md`** (the inherited
   ones are the template's generic copies). Note: this is the Element's **own**
   `CLAUDE.md` at the fork root — distinct from `seed/CLAUDE.md.template`, which
   is the consumer-facing project `CLAUDE.md` that `bin/init` installs into a
   consumer's root (normally inherited unchanged). `seed/` also carries the
   one-time `rasa.lock.json` (stamped at install) and `rasa-deployment.md`.
   *(Optional, §11a: drop a local `ELEMENT_CONTRACT.md` matching your
   `contract_version` (v1.3.0 — the mirror has no v1.4.0 tag): `curl -O https://raw.githubusercontent.com/RasaOS/element-contract/v1.3.0/ELEMENT_CONTRACT.md`.)*

5. **Review the inherited `LICENSE`.** The fork copies `domain-core`'s Apache-2.0
   LICENSE (the public template's). For a **private/proprietary** domain, replace
   or drop it so the license matches the intended visibility + copyright before
   the repo is created.

6. **Fill the scaffolded `CHANGELOG.md` v0.1.0 entry** — set the real date,
   remove the `(TODO: date)` marker and the "TODO before first ship" checklist,
   and describe what the fork actually ships — so the initial tag doesn't publish
   placeholder changelog text.

7. **Stage, then validate.** `check-manifest` inventories only **git-tracked**
   files (`git ls-files`), so **`git add -A` first** — before staging it reports
   "0 tracked files" and passes *vacuously* (proving nothing). Then, from inside
   `elements/domain-<name>/`:
   - **`bin/check-manifest`** — exit **0** is the release gate. Two failure
     classes: **UNREGISTERED** (a tracked file not covered by `element.files`/
     `seed.files` → add it, or an ancestor-directory entry, with the right
     policy) and **DANGLING** (a `from` path that doesn't exist → fix the path or
     drop the entry, e.g. the deleted `SHAPE.md`). Re-run after every content
     change until green.
   - **`bin/check-shape`** — the second, co-equal release gate (STRUCTURE to
     check-manifest's INVENTORY). It errors if any `content/skills/<name>/SKILL.md`
     lacks the required sections `## Behavior contract`, `## Process`,
     `## What NOT to do`, `## Done when`, or has a stub/TODO description, and
     validates rule filenames/H1s + agent frontmatter. (It passes clean on a fork
     with no skills/rules/agents. If you add a skill, update the enumerating note
     on the `content/skills/` `element.files[]` entry or it WARNs — non-fatal.)
   - **§9 basics:** valid JSON, `name` matches the §3 regex, `kind` is `domain`,
     `version` mirrors `VERSION`, no forbidden §8 vocabulary.

8. **First ship (v0.1.0) — gated on explicit confirmation.** This is the manual
   sequence (not `ship-element`, which only bumps an already-shipped element).
   Create the repo, wire `origin`, and push in one step:
   ```
   git add -A && git commit -m "v0.1.0: initial" && git tag v0.1.0
   gh repo create RasaOS/domain-<name> <--public|--private> --source=. --remote=origin --push
   git push --tags
   ```
   Confirm visibility. If `RasaOS/domain-<name>` **already
   exists** (`gh repo view` to check), skip `gh repo create` — just
   `git remote add origin …` (if unset) and push. **Per the workspace rule, don't
   push from the canon-author sandbox** — hand the `gh`/push steps to the user, or
   run only on an explicit go. `bin/ship-element` is for **subsequent** bumps:
   `elements/tenant-core/bin/ship-element domain-<name> --bump <patch|minor|major> --message "..."`
   (preflights clean tree + on `main` + check-manifest; if it fails: commit/stash
   a dirty tree, `git checkout main`, and ensure `git remote -v` shows `origin`).
   It cannot do the initial 0.1.0 ship (no remote, dirty tree). Note it preflights
   only `check-manifest` — run `bin/check-shape` by hand before tagging a bump.

9. **Register.** A new element ship is a **track #2** event.
   - Run `elements/tenant-core/bin/refresh-registry --check`, then
     `--write` (regenerates only the `<!-- BEGIN AUTO: live-elements -->` region
     of `elements/REGISTRY.md` from live disk), or paste the row `ship-element`
     printed.
   - Add the entry to `elements/CHANGELOG.md` (track #2), append a line to
     `canon/AUDIT.md` (the per-action ledger), and add the sibling row to the
     workspace `CLAUDE.md` Zone A table. The workspace `CHANGELOG.md` is
     **track #1** (canon / orchestration work) — a lone domain scaffold doesn't
     touch it.

**Install target (for context):** once shipped, a domain *installs* into a
consumer via `bin/init` — a **manifest/policy-driven** operation, not a blanket
copy. `content/` dirs install only when their `element.files` policy is
`directory-mirror`/`file-replace` (`opt-in` installs nothing); subject content is
conventionally namespaced under **`.claude/<domain>/`** (e.g.
`rasa.domain.alignment` → `.claude/alignment/`). It copies the seed `CLAUDE.md`
to the target **root** (not `.claude/`), stamps `.claude/rasa.lock.json` and
`.claude/rasa-deployment.md`, **generates `.claude/rasa-identity.md`** fresh from
`rasa.identity` (SA-025), and clones a pinned reference copy into
`<target>/kit/<folder>/` that powers the inherited `/sync` + `/promote` skills
(SA-024). (Distinct from *pull*, where the kernel mounts the domain at
`/rasa/modules/<name>/` or the project-embedded `<cwd>/elements/<name>/`.)

## Completion card

After the run, present:

- **Scaffolded:** `elements/domain-<name>/` — `rasa.domain.<name>` v0.1.0,
  `kind: domain`, `shape_pattern: <chosen>`, `capabilities: <collected>`.
- **Follow-ups, each confirmed before acting:**
  1. `rasa.json` filled (`shape_pattern`, `capabilities`, `identity`,
     `contract_version`, `permissions`, `aliases`) + authored content registered
     in `element.files[]`.
  2. `content/` populated per pattern; `SHAPE.md` + template README deleted **and
     deregistered** → fork's own `content/README.md`; root `README.md` +
     `CLAUDE.md` rewritten; `LICENSE` reviewed.
  3. `contract_version` left at the inherited `1.3.0` (fleet convention until the
     v1.4.0 lock); `CHANGELOG.md` v0.1.0 entry filled.
  4. `git add -A`, then `bin/check-manifest` **and** `bin/check-shape` green.
  5. First ship (`git commit`/`tag v0.1.0`/`gh repo create --source=. --push`)
     **or** `bin/ship-element` on a later bump — pushes gated + run outside the
     sandbox.
  6. Registered: `bin/refresh-registry --write` + `elements/CHANGELOG.md`
     (track #2) + `canon/AUDIT.md` line + workspace `CLAUDE.md` Zone A table row.

## Canon

- **Element Contract v1.4.0** — §2 kinds (`domain` = a portable knowledge/toolkit
  foundation), §3 naming (`^rasa\.(domain|module|core|kernel|frontend|recipe|tenant)\..+$`),
  §4 required files (`rasa.json`, `VERSION`, `README.md`, `CLAUDE.md`,
  `CHANGELOG.md`, `.gitignore`; `content/` required for `domain`), §5 required
  rasa.json fields (`$schema`, `contract_version`, `name`, `version`, `kind`),
  §6 optional fields (`capabilities`, `permissions`, `requires`, `aliases`,
  `element.files`, `seed.files`, `rasa.*`, …), §7/§8a install-vs-pull + policies,
  §8 forbidden vocabulary, §9 validation gates, §10 versioning, §11a local
  contract copy.
- **SA-023** — the orchestrator kind was folded into `tenant`; seven kinds now
  (domain, module, core, kernel, frontend, recipe, tenant); modules mount into a
  `domain` or a `tenant` (`requires.parent_kind`).
- **SA-024** — `bin/init` drops a pinned `kit/<element>/` clone that powers
  `/sync` + `/promote`. **SA-025** — `rasa.identity` renders to
  `.claude/rasa-identity.md` at install.
- `domain-core`'s `content/SHAPE.md` — the toolkit / structural / hybrid pattern
  definitions and the `capabilities[]` shapes.

## When NOT to use this skill

- **A mountable capability that extends a parent** (a tenant or a domain) →
  scaffold a **module** with `/create-module` (it seeds
  `requires.parent_kind: ["domain","tenant"]`). Modules replaced the legacy
  "skill" kind.
- **A UI surface, a composition, the shared L1 bones** → those are `frontend` /
  `recipe` / `core`. `new-element` scaffolds all three (`core` is a singleton —
  name forced to `rasa.core`); `tenant` + `kernel` have their own scaffolds and
  are not scaffoldable here. Tenant scaffolding is a separate skill (`/create-tenant` —
  it supersedes the pre-SA-023 `/build-tenant`).
- **Adding content to an existing domain** → edit it in place and
  `bin/ship-element` a bump; don't scaffold a new one.
- **A one-off subject folder inside a consumer's `.claude/`** → that's install
  output, not a new Element.

## What NOT to do

- Don't run `gh repo create`, commit, tag, or push without explicit
  confirmation — and don't push from the canon-author sandbox (hand it to the
  user).
- Don't run `bin/check-manifest` before `git add` — it passes vacuously on an
  unstaged tree ("0 tracked files").
- Don't ship on a red `bin/check-manifest` or `bin/check-shape` — exit 0 on both
  is the gate.
- Don't delete `content/SHAPE.md` without also removing its `element.files[]`
  entry (DANGLING).
- Don't leave `content/SHAPE.md` or the template `content/README.md` in the fork,
  or the template's `rasa.identity` / `LICENSE` unreviewed.
- Don't invent `bin/new-element` flags — it takes only `<kind> <name>
  [--description] [--dry-run] [--workspace]`; everything else is hand-filled.
- Don't use `bin/ship-element` for the initial 0.1.0 ship.
- Don't author `MANIFEST.json` or any forbidden legacy vocabulary (§8).

## Done when

`elements/domain-<name>/` exists as a fresh `domain-core` fork with a filled
`rasa.json` (`shape_pattern`, `capabilities`, `identity`, `contract_version`,
`permissions`, `aliases`, and every authored file covered by `element.files[]`),
its own `content/` populated per the chosen pattern (SHAPE.md + template README
deleted *and deregistered*, PHILOSOPHY + `framework/` + vocabulary in place),
`bin/check-manifest` **and** `bin/check-shape` both exit 0 on a staged tree, the
domain is shipped at v0.1.0 with `origin` wired (or the exact first-ship steps
are handed to the user), and it's registered across the workspace `CLAUDE.md`
table, `elements/REGISTRY.md`, `elements/CHANGELOG.md` (track #2), and
`canon/AUDIT.md`.
