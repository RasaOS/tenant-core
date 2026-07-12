# CHANGELOG — `rasa.tenant.core`

Reverse-chronological. Each entry is a version bump.

---

## 1.4.0 — 2026-07-12

### The `create-*` authoring skills + a co-located tenant scaffolder

The tenant brain gains the standardized, UI-drivable Element-authoring surface
(SA-023 folded the substrate-authoring tooling into the tenant brain):

- **`/create-domain`** + **`/create-module`** — scaffold a new domain / module
  Element the standardized way: interview → `bin/new-element` → fill `rasa.json`
  → register content in `element.files[]` → `git add` → `check-manifest` +
  `check-shape` → first ship → register. Parameterized for a UI; battle-hardened
  via two adversarial audit rounds (fixed real runtime traps: vacuous-green
  `check-manifest` before `git add`, first-ship `origin` wiring, SHAPE.md
  delete-without-deregister, bare-`bin/` path). Instruct declaring
  `contract_version: 1.3.0` (fleet convention until the v1.4.0 lock).
- **`/create-tenant`** + **`bin/build-tenant`** (new) — scaffold a post-SA-023
  co-located tenant: the tenant **root IS the orchestrator** (`rasa.tenant.core`
  at `.claude/` + optional coordination modules), codebase members as gitignored
  siblings (`<ns>-<name>`, auto-prefixed), **no `<ns>-cto` orchestrator member**.
  Supersedes the SA-022-era `/build-tenant` (still shipped in `module-workspace`,
  pending removal). Adversarially reviewed (SA-023/CW + tooling fidelity clean).
- **`bin/new-element`** — doc-only fix: stale docstring examples (retired
  `orchestrator` kind), `~/rAI/rasa-os/` legacy paths, hardcoded contract version;
  logic unchanged (`py_compile` clean).
- CHANGELOG title corrected `rasa.orchestrator.core` → `rasa.tenant.core` (stale
  from the SA-023 rename).

## 1.3.1 — 2026-07-09

### Element identity layer (canon SA-025)

- Added `rasa.identity` ("the foundational RasaOS tenant brain — the orchestrator a tenant installs by default"); `bin/init` generates `.claude/rasa-identity.md` from it every install + stamps project-owned `.claude/rasa-deployment.md`; ships `/whoami`; CLAUDE.md "Who you are" header.

## 1.3.0 — 2026-07-09

### Added generic `/sync` + `/promote` skills + `/kit`-aware `bin/init`

- Added the generic `/sync` (smart-pull) + `/promote` (smart-push) skills
  under `content/skills/`, copied verbatim from the `rasa.domain.core`
  reference. Both are domain-agnostic — they reference only the Element
  contract (lockfile, `pinned_sha`, `overrides`, install policies, the `/kit`
  stash), zero subject knowledge. `/sync` pulls upstream Element changes into
  the consumer's `.claude/` copy; `/promote` pushes portable local
  improvements back upstream as a PR. Both operate against the project's
  persistent `kit/<element>/` clone.
- Replaced `bin/init` with the `/kit`-aware generic installer: at install it
  clones the Element source into `<target>/kit/<folder>/` (the `/kit` stash),
  repoints `origin` at the real upstream repo, and pins the checkout to the
  installed SHA — the reference clone that `/sync` + `/promote` operate on.
- `content/skills/` registration note in `rasa.json` updated to document the
  shipped `/sync` + `/promote`; policy left `opt-in` (scaffold — forks flip to
  `directory-mirror` to install). No change to kind/name identity.

---

## 1.2.0 — 2026-07-09

### Folded to `rasa.tenant.core` + absorbed the substrate tooling (canon SA-023)

- The `orchestrator` kind was folded into `tenant`. This Element (formerly the orchestrator template) becomes rasa.tenant.core — the foundational tenant brain installed into every tenant by default. Absorbed all bin/ tooling from the now-retired rasa.orchestrator.workspace. Renamed rasa.orchestrator.core → rasa.tenant.core; folder orchestrator-core → tenant-core.

---

## 1.1.0 — 2026-05-24 — Pattern 3 (Hybrid) gets a concrete CTO-style example

Minor bump per the post-v1.0 versioning intent ("Minor (1.0.x → 1.1.0)
— additive: ... new shape pattern documented in SHAPE.md"). Adds a
documented canonical example for Pattern 3 (Hybrid) which v1.0.0
left as "none yet."

### Origin

`claude-orchestrator` (`RasaOS/claude-orchestrator`, absorbed
2026-05-22 from `ChazzCoin/claude-orchestrator @ 0f678c5`) was a
pre-canon CTO-orchestrator template that we'd been deferring on
drift-fixing for many sessions. User direction 2026-05-24: "i want
to extract claude-orchestrators core structure into orchestrator.core
and then remove claude-orchestrator."

Rather than canon-shape-migrating claude-orchestrator (a multi-hour
drift fix), we extracted **the pattern it represented** — a
multi-repo coordinator within one operational domain — and
documented it here as the canonical Pattern 3 (Hybrid) example.
Implementation code stays archived on `RasaOS/claude-orchestrator`
(GitHub-archived, read-only); future Hybrid-pattern orchestrators
can read the archive for reference but should fork from this
template + adopt the documented pattern.

### Changed

- **`content/SHAPE.md` Pattern 3 (Hybrid) section** substantially
  expanded:
  - Removed "no examples yet" placeholder
  - Added `claude-orchestrator` as canonical example (extracted v1.1.0)
  - Documented "structure-shaped + domain-shaped" framing — "an
    orchestrator that sits ABOVE a set of related projects, BUT
    all within one operational ownership"
  - Concrete file-layout example showing CTO-orchestrator's `content/`
    (state/sub-repos/, decisions/, proposals/, risks/, governance/,
    templates/sub-repo-*, plus 13 orchestration skills)
  - Documented the **sub-projects registry pattern** (central
    `content/state/sub-repos/` directory with a `bin/register` skill
    that adds new sub-projects; skills like `status`, `audit`,
    `roll-up` iterate over the registry)
  - Capability-namespacing guidance: combine `substrate.*` (cross-repo
    management) + `<vertical>.*` (vertical-specific)
  - `seed/` typically heaviest of the three patterns (substrate
    setup, governance docs)
  - `bin/` often rich (register, refresh, validate-internal-links,
    render-active-notice, etc.)
  - Install posture: YES (installs into a substrate-root umbrella repo)
  - When-to-use + when-NOT-to-use guidance
- **Pattern selection summary table** added at the end of SHAPE.md
  for quick "which pattern do I use" lookup.
- **`rasa.json#version`**: 1.0.0 → 1.1.0
- **`VERSION`**: 1.0.0 → 1.1.0
- **CHANGELOG.md** (this file) — this entry

### What's NOT extracted from claude-orchestrator

The extraction is **documentation only** — the pattern is captured in
SHAPE.md. We deliberately did NOT pull in claude-orchestrator's
implementation:

- **12 bin scripts** (register, refresh, setup, contacts, send-email,
  open-orch-pr, preferences-check, render-active-notice, repo-config,
  validate-internal-links, validate-task-spec) — too CTO-specific.
  Forks adopting Pattern 3 can re-author from the pattern docs +
  consult the archived claude-orchestrator repo for reference.
- **38 bootstrap (seed) templates** (company-profile, tech-vision,
  tech-principles, security-posture, slos, etc.) — too CTO-specific.
  Pattern-3 forks may build their own seed/ heavy with their
  vertical's bootstrap files.
- **32 kit (element) entries** (skill folders, state templates,
  rules) — same. Forks adopt their own implementations of the
  pattern.

This keeps the template lean. Forks understand the SHAPE without
inheriting CTO-specific implementation choices that would have to
be re-customized anyway.

### Backwards compatibility

Existing forks pinning to `rasa.orchestrator.core v1.0.0` are
unaffected. v1.1.0 only adds documentation (no schema change, no
new required files, no new content/, no new bin/). Forks may
opportunistically pull v1.1.0 by running `git pull` against the
upstream + comparing SHAPE.md if they want the Pattern 3 guidance.

### Substrate state implication

The retirement of `claude-orchestrator` (concurrent commit, separate
repo) reduces RasaOS org from 11 → 10 active repos. The archived
`RasaOS/claude-orchestrator` remains accessible read-only as a
historical reference for the CTO-orchestrator pattern this v1.1.0
documents.

---

## 1.0.0 — 2026-05-24 — LOCKED unified shape

First stable lock-down. Parallel work to `rasa.domain.core v1.0.0`
(canon v1.3.0 Phase 2). Authored from the canon-master-orchestrator
session per user direction: "now i want to do same thing for
orchestrators... create a orchestrator.core -> unify them all ->
make sure versioning is setup."

### Why Phase 1+Phase 2 were collapsed

Unlike `rasa.domain.core` (which extracted patterns from two mature
implementations — `rasa.domain.code` toolkit + `rasa.domain.legal`
structural), this orchestrator-core was authored with only **one
canon-conformant orchestrator** (`rasa.orchestrator.workspace v0.2.0`)
plus two empty stubs (`rasa.orchestrator.firm`, `.company`) and one
pre-canon Element (`claude-orchestrator`, awaiting drift-fix in its
own session). With limited comparison data, the stripped→unified
two-step split was collapsed into a single v1.0.0 release. Future
minor bumps remain additive per the post-v1.0 versioning intent.

### Required files (canon §4)

All present:

- `rasa.json` — kind=orchestrator, contract_version=1.3.0
- `VERSION` (1.0.0)
- `README.md` (orient new forks)
- `CLAUDE.md` (per-repo session contract)
- `CHANGELOG.md` (this file)
- `LICENSE` (Apache-2.0, matches claude-kit + domain-core lineage)
- `.gitignore` (macOS + editor cruft)

### Content

- `content/SHAPE.md` — fork-time guide documenting:
  - **Pattern 1 — Domain-shaped** orchestrators (coordinate across N instances within one vertical; example: `rasa.orchestrator.firm`)
  - **Pattern 2 — Structure-shaped** orchestrators (coordinate across siblings across a substrate, orthogonal to domain; example: `rasa.orchestrator.workspace`)
  - **Pattern 3 — Hybrid** (both at once; no canon examples yet)
- `content/{skills,rules,agents}/.gitkeep` — opt-in scaffold subdirs.
  Registered as opt-in in rasa.json#element.files[] so check-manifest
  passes; not installed automatically. Forks populate + change
  policy when adopting.

### Tools (OPTIONAL per canon §4 — forks may delete)

**Key difference from `rasa.domain.core` v1.0.0:** for the
orchestrator template, `bin/init` AND `seed/` are OPTIONAL. Many
orchestrators (especially structure-shaped) don't install — they
operate as a session contract at workspace/tenant root. The template
ships both as starter scaffold; forks delete what they don't need.

- `bin/init` — canonical installer (copied verbatim from
  `rasa.domain.core v1.0.0`). Reads rasa.json, applies element.files[]
  + seed.files[] policies, stamps lockfile.
- `bin/check-manifest` — cross-platform pure-python validator (copied
  from `rasa.domain.core v1.0.0`). Verifies rasa.json is a complete
  inventory of content/ + seed/.
- `seed/CLAUDE.md.template` — per-project Claude session contract
  (placeholder-free; Element identity available via lockfile).
- `seed/rasa.lock.json.template` — canonical lockfile shape with
  substitution placeholders.

### Shape patterns (per content/SHAPE.md)

The template is **shape-agnostic** (`rasa.shape_pattern: "agnostic"`
in rasa.json). Forks pick:

| Pattern | When | Examples |
|---|---|---|
| Domain-shaped | Coordinate across instances within one vertical | `rasa.orchestrator.firm`, `.clinic`, `.company` |
| Structure-shaped | Coordinate across siblings across a substrate | `rasa.orchestrator.workspace` |
| Hybrid | Both | (none yet; future possibility) |

### What's deliberately NOT in v1.0.0

- No orchestration skills/rules/agents (every orchestrator authors
  its own)
- No `bin/lint` (template-stage; not yet canonized)
- No `tasks/` folder (orchestrator's own work-tracking is its
  concern; not part of the shape)
- No `docs/` folder (orchestrator-specific docs are its concern)
- LICENSE is Apache-2.0 — same as domain-core decision

### Versioning intent (post-v1.0)

- **Patch (1.0.x)** — bug fix; no structural change
- **Minor (1.0.x → 1.1.0)** — additive (new canonical bin tool, new
  universal seed template, new shape pattern documented in SHAPE.md,
  new optional scaffold). Forks may adopt opportunistically.
- **Major (1.x.x → 2.0.0)** — breaking; forks REQUIRED to migrate

A fork pinning to `orchestrator-core v1.0.0` is committing to the
v1.0 shape forever; absorbing v1.x minors is optional.

### Smoke-tested

- `bin/check-manifest` → OK (5 tracked files under content/ + seed/,
  all registered as opt-in scaffold or seed entries).
- `bin/init /tmp/oc-test` → clean install (CLAUDE.md +
  .claude/rasa.lock.json copied; opt-in scaffolds correctly NOT
  installed).

### Future considerations (open questions for v1.1+)

When `rasa.orchestrator.firm` + `rasa.orchestrator.company` stubs
become real Elements (Playbook Phase 4b), they'll fork from this
v1.0.0 + adopt the domain-shaped pattern. Patterns that emerge
across those two implementations may absorb into v1.1.0 minor bumps
(e.g., a canonical `seed/INSTANCES.md.template` for matter/patient/
department index, if the pattern proves universal).

When `claude-orchestrator` gets its canon-shape drift-fix (its own
session), it'll likely conform to the structure-shaped pattern AS A
HYBRID (its CTO-style orchestrator pattern has elements of both).
That work may also surface refinements.

The parallel work for the `domain` kind (`rasa.domain.core` v1.0.0)
locked first; this template borrows its bin/init + check-manifest +
seed templates verbatim. If domain-core revises those tools, this
template should consider absorbing the same revision.
