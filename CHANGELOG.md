# CHANGELOG — `rasa.orchestrator.core`

Reverse-chronological. Each entry is a version bump.

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
