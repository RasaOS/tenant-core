# CLAUDE.md — `rasa.orchestrator.core`

> **Who you are (SA-025).** `rasa.tenant.core` — the foundational RasaOS tenant brain — the orchestrator a tenant installs by default. Substrate: **RasaOS**; role: **tenant**. On install `bin/init` renders this into `.claude/rasa-identity.md`; `/whoami` composes the full identity with the project's deployment layer.


Per-repo working contract for Claude sessions opened inside this
folder. Extends `~/.claude/CLAUDE.md` and the workspace
`~/rAI/rasa-os/CLAUDE.md` (which is the `rasa.tenant.rasaos` tenant's
CLAUDE.md); does not override them.

## What you are when you're in this folder

You are working on **`rasa.orchestrator.core`** — the unified
canonical template every `orchestrator` Element follows. The
Element ships **zero orchestration knowledge**; it's the SHAPE every
orchestrator shares.

Two distinct concerns:

- **The template** (this folder) — the canonical shape. Lives at
  `~/rAI/rasa-os/elements/orchestrator-core/`. Pushed to
  `RasaOS/orchestrator-core` (public).
- **The forks** (e.g., `orchestrator-workspace`, future
  `orchestrator-firm`, `orchestrator-company`, `orchestrator-clinic`)
  — concrete orchestrator Elements that adopt this shape and populate
  it with orchestration-specific skills, rules, agents.

When the template's shape changes (future revisions), forked
orchestrators may need to absorb the change. Versioning matters:
v1.0.0 is the lock-down moment; minor bumps additive; major bumps
require fork migration.

## Source of truth

- **`~/rAI/rasa-os/canon/`** — authoritative for everything architectural.
  Current LOCKED is v1.2.0; current WORKING is v1.3.0 IN PROGRESS.
  Spec §6 defines the `orchestrator` kind; ELEMENT_CONTRACT.md §4
  defines required files; §7 defines install policies.
- **`~/rAI/rasa-os/CLAUDE.md`** — workspace orientation (the
  `rasa.tenant.rasaos` tenant's working contract); the role-split
  is locked there.
- **This folder's `README.md`** — what this Element is, how to fork
  it, design history.
- **This folder's `rasa.json`** — formal declaration.
- **`~/rAI/rasa-os/elements/domain-core/`** — the parallel template
  for the `domain` kind. Sister Element; cross-reference both when
  revising shape.

## Key shape difference vs domain-core

Domain-core's v1.0.0 treats `bin/init` as central (toolkit
installation is the typical pattern). Orchestrator-core's v1.0.0
treats `bin/init` AND `seed/` as **OPTIONAL** — many orchestrators
(especially structure-shaped) don't install. They operate as a
session contract at workspace/tenant root.

This means orchestrator forks may legitimately ship without bin/ or
seed/ folders. The template provides starter scaffold; forks
delete what they don't need.

## Don'ts

- **Don't add orchestration-specific content here.** This is the
  template; orchestration-specific material goes in a fork. If a
  piece of content is universal to ALL orchestrators, it MAY land
  here — but that's a future-revision decision, not casual
  authoring.
- **Don't bin/init this Element into itself.** Like every Element
  source repo: `orchestrator-core/.claude/` is for sessions;
  `orchestrator-core/content/` is the source. They don't duplicate.
- **Don't conflate with `rasa.core`** (kind: `core`). Different
  Element. `rasa.core` is the L1 shared bones every Element depends
  on (vocabulary, output styles, stamp definitions).
  `rasa.orchestrator.core` is the template for the `orchestrator`
  KIND. They share a word, not a concern.
- **Don't conflate with `rasa.domain.core`.** Different kind. Same
  role (canonical template) for a different Element category. When
  patterns emerge that apply to BOTH templates, propose canon-level
  adoption (a future canon revision could extract "Element-template
  meta-pattern" into ELEMENT_CONTRACT or a new doc).
- **Don't bump the major version casually.** v1.0.0 is the locked
  starting state. v2.0.0 would force every fork to migrate.

## How a version bump works

- **Patch (1.0.0 → 1.0.1)** — typo fix, README clarification,
  minimal bin tool bug fix. No structural change.
- **Minor (1.0.x → 1.1.0)** — additive: a new canonical bin tool,
  a new universal seed template, new shape pattern documented in
  SHAPE.md, new optional `content/<subdir>/`. Forks may adopt
  opportunistically.
- **Major (1.x.x → 2.0.0)** — breaking shape change. Forks REQUIRED
  to migrate. Avoid without explicit user direction.

Each bump: edit `VERSION`, update `rasa.json#version`, write a
CHANGELOG.md entry. Commit + tag + push. Add a row to
`~/rAI/rasa-os/elements/CHANGELOG.md` (aggregated track #2). Update
`~/rAI/rasa-os/elements/REGISTRY.md` if the row's values change.

## What success looks like for this Element

- A new orchestrator (e.g., `rasa.orchestrator.firm`) can be
  authored by forking this Element + filling content/ — no other
  scaffolding needed.
- The shape decisions are documented (README + this CLAUDE.md +
  CHANGELOG + content/SHAPE.md) so forks understand what they're
  conforming to.
- The two-shape framing (domain-shaped vs structure-shaped) holds
  as more orchestrators get authored; revisions absorb new patterns
  that emerge.
- `orchestrator-workspace` (the current canon-conformant
  orchestrator) aligns with this v1.0.0 shape — proof the
  unification is real, not aspirational.
- When `orchestrator-firm` + `orchestrator-company` stubs become
  real Elements, they fork from here and follow Pattern 1
  (domain-shaped).
