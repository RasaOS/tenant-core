# content/NOTES.md — Authoring roadmap

This file lists candidate skills, rules, and agents for the
`rasa.orchestrator.workspace` Element. Items here are observed
patterns from real workspace work that *could* be generalized into
reusable skill files; they are NOT promises to author.

A skill graduates from this file to `skills/<name>.md` when:
1. The pattern has been observed in real work at least twice.
2. A genuine generalization exists (parameters, edge cases, failure
   modes can be written down).
3. The user invites the authoring — implementation is not done
   proactively at the workspace level.

---

## Candidate skills

| Verb (file would be `skills/<verb>.md`) | What it does | Observed times | Notes |
|---|---|---|---|
| `init-sibling` | When a stub folder needs to become a real Element / consumer repo: git init, install `rasa.domain.code` if it's a consumer, write README + CLAUDE.md from a template, initial commit, `gh repo create RasaOS/<name>`, push. | 2 (kernel — already done by drift-fix session; rasa-console — done 2026-05-22) | The two cases differed: kernel was already a git repo, rasa-console was greenfield. The skill needs to branch on `[ -d .git ]`. |
| `push-multi` | Push N sibling repos to the GitHub org, handling already-exists, wrong-origin, and missing-origin cases. | 1 (this session, but the manual fix-up suggests a real recurring pattern) | The interactive `push-to-rasaos-org.sh` is the first draft. A skill version handles the recovery paths (origin already pointing elsewhere) more robustly. |
| `audit-lockfiles` | Read every sibling's `rasa.lock.json`, compare pinned SHA against current source SHA, surface stale installs. | 1 partial (during today's checks) | First version: just report. Second version: offer `bin/init` re-run for each stale one. |
| `grep-vocabulary` | Scan workspace for banned legacy terms (kit, MANIFEST.json, bootstrap/, foundation.json, manifest contract) and propose canon-form replacements. | 1 (the 2026-05-22 drift fix) | The drift fix was a one-shot. A recurring skill would be much lighter — sweep, report. |
| `reconcile-workspace-docs` | After a session's work, update workspace `CLAUDE.md` + `CHANGELOG.md` to reflect new state. | 2+ (every session has done this implicitly) | The "what to do next" priority queue is the part that drifts fastest. |
| `survey-org` | List all RasaOS repos, their visibility, their default branch, their latest commit SHA, against expected state. | 1 partial | A health-check view of the GitHub org. |
| `version-bump` | For this Element specifically: edit VERSION, update `rasa.json#version`, write CHANGELOG entry, commit. | 0 | Codifies the bump rhythm from this Element's CLAUDE.md. |

## Candidate rules (invariants the orchestrator enforces)

| Rule | What it asserts | Action on violation |
|---|---|---|
| `canon-wins` | When workspace and canon disagree, canon is correct. | Surface to user. Never silently change canon. |
| `vocabulary-lock` | Banned legacy terms (per workspace CLAUDE.md Conventions) do not appear in any sibling. | Surface findings; do not auto-rewrite (rename has historical risk). |
| `no-implementation-code-at-workspace-level` | Claude does not write implementation code in sibling folders at workspace-level invocation. | Reject the action with the role-split memory cite. |
| `lock-pinned-sha-current` | Each consumer sibling's `rasa.lock.json` pinned SHA matches the Element's current source SHA, or has an explicit deferred status. | Surface findings; offer re-install. |
| `commit-coauthor` | Commits from a Claude session include `Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>`. | Reject (or amend) the commit. |
| `private-by-default` | New `RasaOS/*` repos are created with `--private`. | Surface; require explicit opt-in for public. |

## Candidate agents

Maybe none — the orchestrator dispatches to user's existing
`rasa.domain.code` agents per-repo rather than authoring its own.
Revisit if a workspace-level agent emerges that has no per-repo
analogue (e.g., a "cross-repo refactor planner" — but that's
hypothetical).

## What graduates next

Likely first: `reconcile-workspace-docs` — already done implicitly
every session, has a clear shape, generalizes easily.

Likely second: `audit-lockfiles` — small, well-bounded, useful for
detecting drift before it bites.

The rest wait for a second observation.
