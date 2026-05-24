# Orchestrator shape patterns

The `orchestrator` Element kind (canon Spec §6) accepts more than
one internal shape. This document describes the patterns + how to
choose.

When you fork `rasa.orchestrator.core` to author a new orchestrator,
decide upfront which pattern fits — or commit to a hybrid. Then
delete this `SHAPE.md` from your fork and replace it with your
orchestrator's own `content/README.md` describing what's actually
shipped.

---

## Pattern 1 — Domain-shaped orchestrator

**Examples in canon:** `rasa.orchestrator.firm` (law firm), `rasa.orchestrator.clinic`
(medical clinic), `rasa.orchestrator.company` (business).

**What it does:** Coordinates **across N instances WITHIN one vertical**.
A firm orchestrator coordinates across matters; a clinic orchestrator
across patients; a company orchestrator across departments/projects.
The orchestrator lives INSIDE a specific domain.

**Shape:** `content/` contains coordination-pattern subdirs aligned
to the vertical's instance unit:

```
content/
├── skills/           # cross-instance orchestration skills
│   ├── matter-route/      # route work to the right matter
│   ├── cross-matter-summary/
│   └── batch-handoff/
├── rules/            # cross-instance rules
│   ├── matter-isolation.md
│   └── escalation-thresholds.md
├── agents/           # cross-instance dispatch agents
│   └── triage-agent.md
```

**`capabilities[]` shape:** `<domain>.cross-<unit>.*` — e.g.,
`legal.cross-matter.triage`, `health.cross-patient.handoff`,
`engineering.cross-project.standup`.

**Install posture:** TYPICALLY installs via bin/init into a
domain-specific tenant (the firm-home, the clinic-home). Pairs with
a domain Element (rasa.domain.legal + rasa.orchestrator.firm in a
law-firm tenant).

**`seed/`:** typically has CLAUDE.md.template + rasa.lock.json.template
plus possibly an INSTANCES.md.template (firm's matters index,
clinic's patients index).

---

## Pattern 2 — Structure-shaped orchestrator

**Examples in canon:** `rasa.orchestrator.workspace` (the RasaOS
canon-author workspace orchestrator — coordinates substrate-wide
across canon + Elements + consumers).

**What it does:** Coordinates **across siblings ACROSS a substrate**,
orthogonal to the domain question. Spans verticals. The orchestrator
lives ABOVE all domains.

**Shape:** `content/` contains substrate-management subdirs:

```
content/
├── skills/           # substrate-management skills
│   ├── lock-sequence/      # run canon lock sequence
│   ├── element-ship/       # commit + tag + push + REGISTRY + CHANGELOG
│   └── registry-refresh/   # update elements/REGISTRY.md
├── rules/            # substrate invariants
│   ├── canon-edits-go-through-tasks.md
│   └── no-touch-active-projects.md
├── agents/           # substrate-management agents (typically few — usually dispatches via parent Claude)
```

**`capabilities[]` shape:** namespaced families. The canonical
example (`rasa.orchestrator.workspace v0.2.0`) uses `canon.*`,
`substrate.*`, `release.*`, `workspace.*` — 17 capabilities across
four families.

**Install posture:** OFTEN doesn't install via bin/init —
structure-shaped orchestrators operate AS a session contract at the
workspace/tenant root (the workspace `CLAUDE.md` IS the
orchestrator's working contract). The `rasa.json` declares
identity + capabilities; no toolkit gets mirrored into a `.claude/`.

If a structure-shaped orchestrator DOES install (e.g., to bootstrap
a fresh tenant with substrate-management scaffolding), it'll use
bin/init like any other Element.

**`seed/`:** often empty or absent. The "install target" of a
structure-shaped orchestrator is the workspace folder itself, which
already has its own scaffolding.

**`bin/`:** may include orchestrator-specific operational scripts —
e.g., a `bin/audit` for substrate-wide drift sweeps. Optional.

---

## Pattern 3 — Hybrid (CTO-style)

**Canonical example (extracted v1.1.0):** `claude-orchestrator`
(absorbed into `RasaOS/claude-orchestrator` 2026-05-22 from
`ChazzCoin/claude-orchestrator @ 0f678c5`; retired 2026-05-24 after
its CTO-orchestrator pattern was documented here as the canonical
Hybrid example).

**What it does:** Coordinates **across multiple sub-projects/repos
WITHIN a specific operational domain** — typically a company's
multi-repo engineering substrate. Both:

- **Structure-shaped** — spans many repos (sub-projects), tracks
  cross-repo state, registers new repos as the substrate grows.
- **Domain-shaped** — specific to a vertical ("company stack" in
  the CTO case; could equally be "law firm with N matters across N
  repos" or "clinic with N facility repos").

The pattern is "an orchestrator that sits ABOVE a set of related
projects, BUT all within one operational ownership."

**Shape:** mix of substrate-coordination subdirs + vertical-specific
content. Concrete CTO-orchestrator example:

```
content/
├── skills/                          # orchestration skills
│   ├── propose/                     # propose changes across the substrate
│   ├── register/                    # add a new sub-project to the substrate
│   ├── tasks/                       # cross-repo task tracking
│   ├── status/                      # roll-up status across all sub-projects
│   ├── audit/                       # substrate-wide audit
│   ├── review/                      # cross-project PR review coordination
│   ├── onboard/                     # onboard a new team member to the substrate
│   ├── inbox/                       # cross-repo inbox for the principal
│   ├── refresh/                     # refresh substrate state
│   └── roadmap/                     # substrate-wide roadmap rollup
├── rules/                           # cross-substrate rules
│   └── orchestrator-rules.md
├── state/                           # cross-repo state (the "substrate registry")
│   └── sub-repos/_template.md       # template for registering a sub-project
├── templates/                       # vertical-specific notice/advertisement templates
│   ├── sub-repo-notices/
│   ├── sub-repo-shared/
│   └── sub-repo-advertisement/
├── decisions/                       # decision log
├── proposals/                       # proposal staging
├── risks/                           # risk register
├── governance/                      # governance docs
├── features/                        # feature roadmap
├── migrations/                      # cross-repo migration tracking
├── pr-reviews/                      # PR review state
├── incidents/                       # incident log
└── reviews/                         # review cycles
```

**Sub-projects registry** (the substrate concept):

A central `content/state/sub-repos/` directory holds one .md per
registered sub-project. A `bin/register` script (or equivalent
skill) adds new sub-projects to this registry. Skills like
`status`, `audit`, `roll-up` iterate over the registry to operate
across the substrate.

This is the "macro architectural truth across one operational
domain's stack" pattern. The orchestrator owns the registry; the
sub-projects own their own implementations.

**`capabilities[]` shape:** combine substrate-management caps with
vertical-specific caps:
- substrate.*: `substrate.sub-repo-registration`, `substrate.cross-repo-audit`, `substrate.roll-up`
- <vertical>.*: `company.tech-vision`, `company.architectural-truth` (CTO case); or `legal.cross-matter.*`, `health.cross-patient.*` for vertical-specific hybrids

**`seed/`:** typically the heaviest of the three patterns. The
seeded files set up the substrate's own scaffolding:

- `company-profile.md.template`, `tech-vision.md.template`,
  `tech-principles.md.template` (vertical-specific)
- `roadmap.md.template`, `open-questions.md.template`,
  `decision-log.md.template`, `AUDIT.md.template`
  (universal-orchestrator material that any Pattern-3 fork may want)
- `repos.json.template`, `integrations.json.template`,
  `contacts.json.template` (substrate registry seeds)
- `compliance.md.template`, `security-posture.md.template`,
  `slos.md.template` (governance — CTO-specific)

**`bin/`:** often rich — operational scripts for substrate
management:

- `bin/register` — add a new sub-project to the registry
- `bin/refresh` — refresh substrate state across all sub-repos
- `bin/validate-internal-links`, `bin/validate-task-spec` — gates
- `bin/render-active-notice` — generate cross-repo notices
- `bin/send-email`, `bin/contacts` — communication (CTO-specific)

**Install posture:** YES — Pattern 3 orchestrators install into a
substrate root (e.g., the company's umbrella repo) where the
registry + state files live. Different from Pattern 2
(structure-shaped) which often doesn't install at all.

**When to use it:** when the orchestrator needs to coordinate
**multiple sub-projects** all owned by the same operational entity
(a company, a firm with many matters in separate repos, a clinic
with many facility repos). Both structure (across repos) AND
domain (within a vertical's operational scope).

**When NOT to use it:**
- If purely coordinating instances within ONE repo → use Pattern 1 (Domain-shaped)
- If coordinating across the entire substrate above all verticals → use Pattern 2 (Structure-shaped)
- If you don't have multiple repos to span → either Pattern 1 or simpler

---

## Pattern selection summary

| If your orchestrator... | Use Pattern |
|---|---|
| Coordinates instances (matters/patients/projects) within one vertical, one repo | 1 — Domain-shaped |
| Coordinates substrate-wide across many domains | 2 — Structure-shaped |
| Coordinates many sub-repos within one operational domain (a company, a firm with multi-repo) | 3 — Hybrid (CTO-style) |

---

## Which pattern does `rasa.orchestrator.core` v1.0.0 commit to?

**None.** The template is shape-agnostic. It ships:

- `content/skills/`, `content/rules/`, `content/agents/` as **opt-in
  scaffold** (registered in rasa.json with `policy: opt-in` so
  check-manifest passes but they don't install automatically).
- `content/SHAPE.md` (this file) explaining the choice.

What the template enforces is the **Element primitive** (rasa.json
+ content/ + seed/ + bin/init) per canon, not the internal shape
of `content/` and not the install posture.

**Key difference from domain-core v1.0.0:** for the orchestrator
template, `bin/init` and `seed/` are OPTIONAL. Many orchestrators
(especially structure-shaped) don't install. Domain-core's template
treats bin/init as central to the install pattern; orchestrator-core
treats it as one valid option among many.

---

## How to fork

```sh
# 1. Clone the template
git clone https://github.com/RasaOS/orchestrator-core.git orchestrator-<yourname>
cd orchestrator-<yourname>

# 2. Re-point git remote
git remote set-url origin git@github.com:<your-org>/orchestrator-<yourname>.git

# 3. Edit rasa.json
#    - name: rasa.orchestrator.<yourname>
#    - version: 0.1.0 (reset)
#    - description: <your orchestrator description>
#    - capabilities[]: namespaced family enumeration
#    - rasa.shape_pattern: "domain-shaped" | "structure-shaped" | "hybrid"

# 4. Decide install posture:
#    - If installs: keep bin/init + seed/, populate
#    - If doesn't install: delete bin/init + seed/, remove
#      element.files[] + seed.files[] from rasa.json

# 5. Populate content/ per your chosen pattern
#    - domain-shaped → skills/{matter-route, …}, rules/{matter-isolation, …}
#    - structure-shaped → skills/{lock-sequence, …}, rules/{substrate invariants}

# 6. Delete this SHAPE.md, replace with content/README.md

# 7. Bump version, commit, tag v0.1.0, push
```

---

## See also

- Canon Spec §6 — the `orchestrator` kind definition
- Canon ELEMENT_CONTRACT.md §2 — kinds table
- `~/rAI/rasa-os/elements/orchestrator-workspace/` — Pattern 2 reference (structure-shaped, the canon master orchestrator)
- `~/rAI/rasa-os/elements/orchestrator-firm/` — Pattern 1 stub (domain-shaped, future legal-firm orchestrator)
- `~/rAI/rasa-os/elements/orchestrator-company/` — Pattern 1 stub (domain-shaped, future company orchestrator)
- `~/rAI/rasa-os/elements/domain-core/content/SHAPE.md` — parallel shape document for the `domain` kind
