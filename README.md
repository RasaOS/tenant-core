# RasaOS Orchestrator · Core

**Canonical name:** `rasa.orchestrator.core`
**Repo / folder:** `orchestrator-core`
**Kind:** `orchestrator` (canon Spec §6)
**Contract:** Element Contract v1.3.0
**Version:** 1.0.0 (LOCKED unified shape)
**Status:** First stable lock-down. Shape-agnostic. Forks may pin here; subsequent minor bumps are additive.

## What this is

The unified canonical template every `orchestrator` Element follows.
Fork this when authoring a new orchestrator and decide upfront which
shape pattern fits (see `content/SHAPE.md`):

- **Domain-shaped** — coordinates across N instances WITHIN one
  vertical (`rasa.orchestrator.firm`, `.clinic`, `.company`).
- **Structure-shaped** — coordinates across siblings ACROSS a
  substrate, orthogonal to domain (`rasa.orchestrator.workspace`).
- **Hybrid** — both at once. Allowed; document clearly.

This Element holds **zero orchestration knowledge** — it's the SHAPE
every orchestrator shares.

## What this isn't

- **Not `rasa.core`** (kind: `core`) — that's the L1 shared bones
  every Element depends on (vocabulary, output styles, stamp
  definitions). Different concerns; both named "core" for canonical
  positioning.
- **Not `rasa.domain.core`** — the parallel template for the
  `domain` kind. Same role, different kind. Sister Elements.

## Key shape note — bin/init + seed/ are OPTIONAL for orchestrators

Unlike domains (where `bin/init` is typically required because the
toolkit installs into consumer projects), **many orchestrators don't
install**. Structure-shaped orchestrators (`rasa.orchestrator.workspace`)
often operate AS a session contract at workspace root — the
workspace `CLAUDE.md` IS the orchestrator's working contract. No
toolkit gets mirrored into a `.claude/`.

The template ships `bin/` + `seed/` as starter scaffold so forks
that DO install have ready material. Forks that don't install can
delete both folders entirely + remove the `element.files[]` +
`seed.files[]` arrays in their `rasa.json`.

## File layout (v1.0.0 locked)

```
orchestrator-core/
├── rasa.json              # Connection Contract — kind=orchestrator, contract_version=1.3.0
├── VERSION                # semver, currently 1.0.0
├── README.md              # this file
├── CLAUDE.md              # per-repo working contract for sessions on this template
├── CHANGELOG.md           # version history
├── LICENSE                # Apache-2.0 (matches claude-kit + domain-core lineage)
├── .gitignore             # macOS + editor cruft
├── bin/                   # OPTIONAL — forks delete if not installing
│   ├── init               # canonical installer (same as domain-core)
│   └── check-manifest     # cross-platform pure-python validator (same as domain-core)
├── content/
│   ├── SHAPE.md           # explains domain-shaped vs structure-shaped patterns; fork-time guide
│   ├── skills/.gitkeep    # orchestration skill scaffold (opt-in)
│   ├── rules/.gitkeep     # orchestration rules scaffold (opt-in)
│   └── agents/.gitkeep    # Claude subagents scaffold (opt-in)
└── seed/                  # OPTIONAL — forks delete if not installing
    ├── CLAUDE.md.template       # per-project Claude session contract (placeholder-free)
    └── rasa.lock.json.template  # canonical lockfile shape (substituted at install)
```

## How to fork

See `content/SHAPE.md` "How to fork" section for the step-by-step.
TL;DR:

1. `git clone` → re-point remote
2. Edit `rasa.json` (name + version + description + capabilities[]
   + `rasa.shape_pattern`)
3. Decide install posture: keep or delete `bin/` + `seed/`
4. Populate `content/` per your pattern (domain-shaped or
   structure-shaped)
5. Delete `content/SHAPE.md`; replace with `content/README.md`
6. Bump VERSION, tag, push

## Why this Element exists (the design pass)

Phase 2 of `rasa.domain.core` (canon v1.3.0) extracted the unified
shape across `rasa.domain.code` (toolkit pattern) + `rasa.domain.legal`
(structural pattern). The parallel work for `orchestrator` kind
started with only **one** canon-conformant implementation
(`rasa.orchestrator.workspace`) plus two empty stubs
(`rasa.orchestrator.firm`, `.company`) and one pre-canon Element
(`claude-orchestrator`, awaiting drift-fix in its own session).

With limited comparison data, the Phase 1 → Phase 2 split (stripped
template → unified template) used for domains was collapsed: this
Element ships at v1.0.0 directly. Future minor bumps remain additive
per the post-v1.0 versioning intent.

The two-shape framing (domain-shaped vs structure-shaped) is
extrapolated from canon Spec §6 + the canon master orchestrator
(`rasa.orchestrator.workspace v0.2.0`) documented identity. When
domain-shaped orchestrators are actually authored
(`rasa.orchestrator.firm`, etc.), a v1.1.0 revision may absorb
patterns that emerge.

## Required files per canon ELEMENT_CONTRACT §4

All present:

| File | Status | Notes |
|---|---|---|
| `rasa.json` | ✓ | kind=orchestrator |
| `VERSION` | ✓ | 1.0.0 |
| `README.md` | ✓ | this file |
| `CLAUDE.md` | ✓ | per-repo session contract |
| `CHANGELOG.md` | ✓ | v1.0.0 entry |
| `.gitignore` | ✓ | |
| `content/` | ✓ | with SHAPE.md + scaffold subdirs |
| `seed/` | ✓ (optional per canon) | forks may delete |
| `bin/init` | ✓ (optional per canon) | forks may delete |
| `bin/check-manifest` | ✓ (recommended) | useful for tagging discipline |

## Versioning intent (post-v1.0)

- **Patch (1.0.x)** — typo fix, README clarification, bin tool bug fix.
- **Minor (1.0.x → 1.1.0)** — additive: a new canonical bin tool, a new universal seed template, a new optional `content/<subdir>/` scaffold, a new shape pattern documented in SHAPE.md. Forks may adopt opportunistically.
- **Major (1.x.x → 2.0.0)** — breaking shape change. Forks REQUIRED to migrate. Avoid without explicit user direction.

## See also

- Canon Spec §6 (Element kinds — `orchestrator` definition)
- Canon ELEMENT_CONTRACT.md (every Element follows this)
- `~/rAI/rasa-os/elements/domain-core/` (the parallel template for the `domain` kind)
- `~/rAI/rasa-os/elements/orchestrator-workspace/` (Pattern 2 reference — structure-shaped, the canon master orchestrator)
- `~/rAI/rasa-os/elements/REGISTRY.md` (live snapshot of every Element under workspace management)

## License

Apache 2.0. Matches the lineage used by `rasa.domain.core` and the
historical claude-kit/claude-legal-kit upstreams.
