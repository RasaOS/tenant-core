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

## Co-located tenant workspace (SA-022)

A tenant MAY be a **co-located workspace** (canon v1.4.0 CW-001 — a
flavor, not a mandate; the other tenant flavor is the SA-019
holding-folder / canon-author-root model). A co-located workspace is a
thin container directory (just `rasa.json` + `CLAUDE.md`) holding
**member repos** as co-located **siblings** inside it. Each member is
its own git repo with its own remote, named `<tenant-namespace>-<name>`.
**When a tenant uses this flavor, the orchestrator instance IS one of
those member repos** (`<ns>-cto`, etc.), co-located alongside the
codebase members it coordinates — it does NOT clone them into a
gitignored `repos/<name>/` subdir or a `.rasa/holding/` folder:

```
vsi-ops/            # the tenant — thin: rasa.json + CLAUDE.md, gitignores every member
├── vsi-cto/        # the ORCHESTRATOR INSTANCE — itself a member repo
├── vsi-web/        # codebase member repo
├── vsi-ios/        # codebase member repo
└── vsi-devops/     # infra codebase member (rasa.domain.devops installed into it)
```

The locks (canon v1.4.0 CW-001..CW-006):

- **CW-001** — a tenant MAY be a co-located workspace; each member is
  its own git repo; the tenant gitignores every member. Git-per-repo
  IS the sync (full clones, no holding folder).
- **CW-002** — member naming `<tenant-namespace>-<name>`, folder + repo
  dashed.
- **CW-003** — the orchestrator instance IS a member repo (`<ns>-cto`),
  co-located with the codebases it coordinates — NOT at the tenant
  root, NOT in a `.rasa/holding/` folder.
- **CW-004** — the tenant declares `tenant.members[] = { name, role,
  git_remote, installed_element? }`.
- **CW-005** — orchestrators coordinate co-located members and MAY
  create + register new ones (scaffold `<ns>-<name>`, git init,
  `gh repo create`, register). Applies to **all** orchestrator
  patterns below.
- **CW-006** — canon Elements install INTO member repos (codebase
  member ← `rasa.domain.*`; orchestrator member ← `rasa.orchestrator.*`).
  Members are install targets; Elements are sources.

**What this means for the patterns:** in a co-located tenant, wherever
a pattern below speaks of "sub-projects," "sub-repos," or a
"recommended sibling layout," that resolves to **co-located member
repos** — the orchestrator reaches its members at the relative path
`../<ns>-<name>` from its own member root (`<ns>-cto/`). The "registry"
a Pattern-3 orchestrator owns is the tenant's `members[]` plus its own
`state/` records of those siblings. (Multi-repo coordination is
Pattern-3 territory; a Pattern-1 orchestrator coordinates instances
*within* its own member repo — see below.)

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
domain-specific tenant (the firm-home, the clinic-home). In a
co-located tenant (SA-022) the orchestrator is a member repo
(`<tenant>/<ns>-firm/`) and `rasa.domain.legal` installs INTO it (or
into a paired codebase member), pairing domain + orchestrator inside
one law-firm tenant. It coordinates its instances (matters, patients)
*within* its own member repo — instances are not separate repos. If
the firm's work genuinely spans multiple repos, that's the Pattern-3
hybrid (co-located member siblings at `../<ns>-<name>`), not Pattern 1.

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
structure-shaped orchestrators operate AS a session contract. Two
seats, depending on the tenant flavor it coordinates:

- **At the tenant/workspace root** (SA-019 canon-author flavor) — the
  orchestrator's contract IS the tenant-root `CLAUDE.md`, coordinating
  the Elements/repos checked out under that root. This is the
  `rasa.orchestrator.workspace` / `rasa.tenant.rasaos` reference case,
  where members are full clones directly under the root (not `<ns>-`
  siblings), synced git-per-repo.
- **As a co-located member** (`<ns>-cto/`) in a co-located tenant
  (SA-022) — it coordinates its member siblings at `../<ns>-<name>`
  and MAY create + register new members (CW-005).

Either way the `rasa.json` declares identity + capabilities. A
structure-shaped orchestrator that DOES install (e.g., to bootstrap a
fresh tenant with substrate scaffolding) uses bin/init like any other
Element — under SA-022 the install target is its own member repo.

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

**Member registry** (the substrate concept):

Under SA-022 the "sub-projects" ARE the tenant's co-located member
repos. A central `content/state/sub-repos/` directory holds one .md
per registered member (mirroring the tenant's `members[]`), each a
sibling of the orchestrator at `../<ns>-<name>`. A `bin/register`
script (or equivalent skill) adds new members: scaffold
`<ns>-<name>`, git init, `gh repo create`, register (CW-005). Skills
like `status`, `audit`, `roll-up` iterate over the registry to
operate across the co-located siblings.

> **Follow-up (code):** `bin/register` currently clones into a
> gitignored `repos/<name>/` subdir. To match SA-022 it must instead
> create the member as a co-located sibling at `../<ns>-<name>` (its
> own repo + remote), not a subdir clone. Note as a bin/ change, not
> yet done.

This is the "macro architectural truth across one operational
domain's stack" pattern. The orchestrator owns the registry; the
members own their own implementations.

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

- `bin/register` — add a new co-located member to the registry
  (scaffold `<ns>-<name>` sibling + git init + `gh repo create`; see
  the code follow-up above)
- `bin/refresh` — refresh substrate state across all sub-repos
- `bin/validate-internal-links`, `bin/validate-task-spec` — gates
- `bin/render-active-notice` — generate cross-repo notices
- `bin/send-email`, `bin/contacts` — communication (CTO-specific)

**Install posture:** YES — under SA-022 a Pattern 3 orchestrator
installs into its own member repo (`<ns>-cto/`), co-located inside
the tenant alongside the codebase members it coordinates (NOT at the
tenant root). Its registry + state files live in that member; the
codebase members it spans are siblings at `../<ns>-<name>`, each with
its own remote. Different from Pattern 2 (structure-shaped) which
often doesn't install at all.

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
