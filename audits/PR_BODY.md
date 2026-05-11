# Harden agent prompts: 9-pattern RCA from real-world failure evidence

## Summary

- Conducted a 5-Whys root-cause analysis on nine recurring failure patterns
  observed in a production mobile project. All patterns are grounded in commit
  history and status-document evidence.
- Applied localized, generic guardrail insertions to nine existing agent files.
  No new agents created. No existing behaviour removed.
- Added `audits/2026-05-naamjap-rca.md` (evidence + drilldowns),
  `audits/stub-declaration-contract.md` (the new conventions), and this PR body.

## Patterns addressed

| ID | Pattern | Agents touched |
|----|---------|---------------|
| P1 | UI control shipped without backing implementation | `code-reviewer`, `mobile-developer`, `full-stack-developer`, `product-manager` |
| P2 | Status entry marked complete without a traceable artifact | `documentation-expert`, `product-manager` |
| P3 | User preference written to storage, never read by any code path | `code-reviewer`, `full-stack-developer` |
| P4 | New data entity introduced; downstream display surfaces not updated | `code-reviewer`, `architect-review`, `full-stack-developer` |
| P5 | UI visual properties not verified across all color-scheme / surface variants | `ui-designer`, `code-reviewer` |
| P6 | Feature ships on one platform; other target platforms not recorded in status | `mobile-developer`, `qa-expert` |
| P7 | Deployment prerequisite documented as manual; CI gate is advisory-only (warns, not fails) | `deployment-engineer`, `test-automator` |
| P8 | Mock-only unit suite passes; cross-surface path broken against real dependencies | `test-automator`, `qa-expert` |
| P9 | Code review approved PR without checking P1–P8 | `code-reviewer` |

## What changed in each agent file

**`agents/quality-testing/code-reviewer.md`** — Added new checklist section
"Integration Surface & Correctness" covering P1 (UI-implementation pairing),
P3 (preference read sites), P4 (cross-surface propagation), P5 (visual context
coverage), and P9 (explicit sign-off checklist naming all eight patterns).

**`agents/development/mobile-developer.md`** — Added "Integration Surface
Guardrails" section: P1 (stub declaration required for phased UI), P6
(cross-platform parity must be recorded, not omitted).

**`agents/specialization/documentation-expert.md`** — Added "Status Accuracy
Contract": P2 (traceable artifact required for ✅, challenge unverified rows).

**`agents/business/product-manager.md`** — Added "Status Integrity Rules":
P2 (artifact-backed completion criteria), P1 (stub declaration required in
status entries for phased features).

**`agents/quality-testing/architect-review.md`** — Added "Cross-Surface
Propagation" bullet to "Key Areas of Focus": P4 (enumerate all surfaces
consuming a new entity; flag uncovered surfaces as incomplete rollout).

**`agents/development/ui-designer.md`** — Added "Visual Context Completeness"
to Constraints: P5 (every component verified across all color-scheme and
surface variants; spec note is not a substitute for verification).

**`agents/quality-testing/qa-expert.md`** — Added two Guiding Principles:
P6 (test plans must list all platforms, including unimplemented ones) and P8
(cross-surface integration test required for every new entity, against real
dependencies).

**`agents/infrastructure/deployment-engineer.md`** — Added "No Advisory-Only
Gates" Guiding Principle: P7 (prerequisites must trigger hard-fail, not warn;
if automation impossible, document a time-bounded override process).

**`agents/quality-testing/test-automator.md`** — Added two Guiding Principles:
P7 (CI gate completeness — escalate warns to fails) and P8 (cross-surface
integration coverage — full path through real dependencies required).

**`agents/development/full-stack-developer.md`** — Added three Technical
Standards: P1 (UI-implementation pairing), P3 (preference read sites), P4
(cross-surface propagation with explicit follow-up task if deferred).

## What was not changed

- No agent was created. A "Status Verification Agent" was proposed in the RCA
  under "Future agent proposals" for discussion after the guardrails are
  validated.
- No existing agent behaviour was removed or weakened.
- All insertions are localized; no section was rewritten wholesale.
- Every insertion ends with `<!-- added: audit/naamjap-rca -->` for traceability.

## Test plan

- [ ] Read each edited agent file end-to-end; confirm new clauses do not
  contradict existing behaviour.
- [ ] Apply the mental swap test to each inserted clause: swap the domain to
  "a recipe app" or "a fitness tracker" — confirm the clause retains meaning.
- [ ] Confirm no project-specific noun (app name, domain-specific feature names)
  appears in any agent file.
- [ ] Confirm every insertion ends with `<!-- added: audit/naamjap-rca -->`.
- [ ] Confirm `audits/2026-05-naamjap-rca.md` has a grounded evidence block for
  each of P1–P9.
- [ ] Confirm `audits/stub-declaration-contract.md` cross-references all agent
  files that carry each convention.

---

🤖 Generated with [Claude Code](https://claude.ai/claude-code)
