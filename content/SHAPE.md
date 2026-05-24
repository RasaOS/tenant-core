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

## Pattern 3 — Hybrid

**Examples in canon:** none yet. A plausible future shape: an
orchestrator that BOTH coordinates across instances within a vertical
AND publishes cross-vertical primitives (a multi-vertical company
orchestrator that handles both internal coordination + external
substrate hooks).

**Shape:** mix of Pattern 1 subdirs (matter-route/, etc.) plus
Pattern 2 subdirs (substrate-hooks/). Allowed; no contract
violation. Document clearly in the fork's `content/README.md`.

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
