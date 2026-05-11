# Root-Cause Analysis — NaamJap Project Audit
**Branch:** `audit/naamjap-rca-generic-hardening`  
**Date:** 2026-05-10  
**Evidence sources:** `~/naam_jaap/docs/IMPLEMENTATION_STATUS.md`, `git -C ~/naam_jaap log`

> This document is the evidence record for the agent hardening on this branch.
> All project-specific nouns (app name, feature names, framework identifiers) are
> confined to this file. The agent prompts this RCA produced are entirely generic.

---

## Pattern index

| ID | Pattern | Confirmed? | Agent files touched |
|----|---------|-----------|-------------------|
| P1 | UI-only stub shipped as complete | ✅ Strong | `code-reviewer.md`, `mobile-developer.md`, `full-stack-developer.md`, `product-manager.md` |
| P2 | Stale ✅ in status / docs | ✅ Confirmed | `documentation-expert.md`, `product-manager.md` |
| P3 | Setting / preference with no consumer | ✅ Confirmed | `code-reviewer.md`, `full-stack-developer.md` |
| P4 | Feature built in isolation; downstream surfaces not updated | ✅ Strong | `code-reviewer.md`, `architect-review.md`, `full-stack-developer.md` |
| P5 | Non-adaptive UI; contrast not verified across visual contexts | ✅ Process gap | `ui-designer.md`, `code-reviewer.md` |
| P6 | Cross-platform parity gap shipped silently | ✅ Confirmed | `mobile-developer.md`, `qa-expert.md` |
| P7 | Manual config step in docs, not enforced by CI | ✅ Confirmed | `deployment-engineer.md`, `test-automator.md` |
| P8 | Tests green, feature broken end-to-end | ✅ Confirmed | `test-automator.md`, `qa-expert.md` |
| P9 | Reviewer didn't catch P1–P8 | ✅ Confirmed | `code-reviewer.md` |

---

## P1 — UI-only stub shipped as complete

### Evidence

- `docs/IMPLEMENTATION_STATUS.md:210` — iCloud Sync (🟧): "Settings UI present
  (toggle + Force Sync Now button + last-synced timestamp). Transport not wired —
  there's no actual NSUbiquitousKeyValueStore / CloudKit code path."
- `docs/IMPLEMENTATION_STATUS.md:211` — Android Cloud Sync (🟧): "toggle present,
  no transport."
- `docs/IMPLEMENTATION_STATUS.md:258` — Online library refresh: marked `✅ (stub)`;
  `refreshFromRemote()` is an explicit no-op.
- `docs/IMPLEMENTATION_STATUS.md:97-98` — Analytics opt-out toggle and push-topic
  opt-out toggle both persist to preferences; neither gates the SDK calls they imply.
- Commit `da3243e` — "add sync settings UI" shipped sync controls with zero transport
  code; merged via PR `76ac3a7` without flagging the gap.

### 5-Whys

1. **Why** did users see a sync toggle that did nothing?  
   → UI was shipped before the transport layer was built.
2. **Why** was UI-without-transport merged?  
   → The PR author treated UI visibility as a useful placeholder; the reviewer did
   not distinguish "stub" from "shipped."
3. **Why** did no process flag this distinction?  
   → The Definition of Done checked test passage and linting, not whether each
   visible control had a backing implementation.
4. **Why** was no explicit stub contract required?  
   → Agent prompts for full-stack and mobile developers did not mandate that a
   stub be declared and labelled — the silent default was "done."
5. **Root cause (instruction level):** The completion criteria in developer and
   reviewer agent prompts do not require tracing every user-visible control to
   a real implementation. Stubs are undeclared and therefore invisible to review.

### Diff summary

Added "UI-Implementation Pairing" clause to `code-reviewer.md` (review checklist),
`mobile-developer.md` (guardrails section), `full-stack-developer.md` (technical
standards), and `product-manager.md` (stub declaration rule). Each clause requires
that any UI control implying a feature must trace to a real implementation, or be
explicitly declared as a stub with the missing component named.

---

## P2 — Stale ✅ in status / docs

### Evidence

- `docs/IMPLEMENTATION_STATUS.md:204` — Time Calculator (US-V1-033): explicit note
  "Stale ✅ correction (2026-05-07): ARB strings exist, but no screen, route, or
  settings entry tile is implemented."
- Commit `b71e5a0` — "correct status doc": ✅→⏳, shipped count adjusted 36→35,
  pending count 44→45. The correction was manual and reactive.
- The original ✅ was placed because translation strings existed — copy resources
  treated as equivalent to a working feature.

### 5-Whys

1. **Why** was a feature marked ✅ when only copy strings existed?  
   → The author applied ✅ when partial work (strings) was done, conflating
   "touched the feature" with "feature is complete."
2. **Why** didn't review catch this?  
   → No checklist item required status entries to cite a concrete implementation
   artifact (route, endpoint, file path).
3. **Why** was a string resource treated as evidence of completion?  
   → The status document's legend defined ✅ loosely ("meets acceptance criteria")
   without requiring a pointer to where the implementation lives.
4. **Why** was the legend definition not enforced?  
   → Documentation and PM agent prompts did not require traceable artifacts as
   a condition for marking completion.
5. **Root cause:** The documentation and PM agents lack a "traceable artifact"
   rule — copy, config, or migration stubs can silently satisfy a ✅ mark.

### Diff summary

Added "Traceable Completion" rule to `documentation-expert.md` and
"Artifact-Backed Completion" rule to `product-manager.md`. Both require a ✅
row to cite a file path, route, or commit reference; copy or string resources
alone are disqualified.

---

## P3 — Setting / preference with no consumer

### Evidence

- `docs/IMPLEMENTATION_STATUS.md:97` — "Help improve [app]" analytics toggle:
  "persists; `firebase_analytics` SDK reads not yet gated by it."
- `docs/IMPLEMENTATION_STATUS.md:98` — Push-topic opt-out toggle: "persists;
  topic-subscribe code not wired."
- Both toggles write a value to SharedPreferences that the SDK never reads,
  making the user's opt-out functionally meaningless — a privacy regression
  even though no exception is thrown and no test fails.

### 5-Whys

1. **Why** does the analytics opt-out have no effect?  
   → The SDK integration was deferred but the UI toggle was not — so the toggle
   persists a value that nothing reads.
2. **Why** wasn't the missing read site caught in review?  
   → Reviewers checked that the toggle wrote to storage; they did not verify
   that anything read from storage to gate the SDK.
3. **Why** is a write without a read acceptable by default?  
   → Developer agent prompts have no requirement to identify and document the
   read site when a preference is introduced.
4. **Why** does this matter beyond UX?  
   → A preference that controls data collection but is never read is a silent
   privacy defect — opt-out is broken without any visible signal.
5. **Root cause:** No agent prompt requires that every preference write be
   paired with a documented read site at implementation time.

### Diff summary

Added "Preference Read Sites" clause to `code-reviewer.md` and
`full-stack-developer.md`. The clause requires identifying the code path that
reads and branches on a stored preference as a condition of completion.

---

## P4 — Feature built in isolation; downstream surfaces not updated

### Evidence

- Commit `2fa2869` — Dual-counter mode shipped with Insights card updated;
  History reads and streak SQL still used the single-counter column only,
  making dual-mode practice days invisible in streaks and History.
- Commit `415eb2c` — Follow-up "parity across counter / streak / history" patch
  (3 days later): streak SQL updated to `jap_count > 0 OR mantra_count > 0`,
  History switched to streaming query including the second counter column.
- Commit `020a216` — "integration regression net for dual-counter divergence":
  integration tests added specifically because the bug escaped. Commit message
  explicitly names the divergence as a class of bug to prevent recurrence.

### 5-Whys

1. **Why** were History and streak calculations not updated when the dual-counter
   was introduced?  
   → The feature was scoped to "add the new counter UI + Insights card." History
   and streak were not in the mental checklist for the feature.
2. **Why** wasn't an incomplete rollout caught before merge?  
   → The unit tests covered the new counter surface in isolation; no test
   crossed from the counter to History or streak to verify the new data appeared.
3. **Why** was no cross-surface check part of the Definition of Done?  
   → Developer and architect prompts required tests for the changed surface but
   not an explicit enumeration of all surfaces that consume the new data.
4. **Why** is downstream surface enumeration not standard practice?  
   → Feature scoping documents describe what to build, not which existing
   surfaces must be updated to remain consistent.
5. **Root cause:** No agent prompt requires the implementer to enumerate all
   downstream surfaces when a new data entity or capability is introduced.

### Diff summary

Added "Cross-Surface Propagation" clause to `code-reviewer.md`,
`architect-review.md`, and `full-stack-developer.md`. Each clause requires
enumerating all surfaces that display or aggregate a new entity, and flags
any unupdated surface as an incomplete rollout.

---

## P5 — Non-adaptive UI; contrast not verified across visual contexts

### Evidence

- `docs/design-spec.md:2056` — "Run wallpaper backgrounds through contrast
  checker at multiple bright spots. The 0.45 dark overlay must be sufficient;
  increase to 0.55 if any wallpaper fails." — This is a manual recommendation,
  not an automated or gated check.
- `docs/design-spec.md:2037-2039` — WCAG AA contrast ratios are specified but no
  CI step enforces them.
- No confirmed visual defect in the evidence; the failure mode is process
  exposure: the verification step exists only as a doc note, skippable under
  deadline pressure.

### 5-Whys

1. **Why** might a shipped component fail contrast requirements?  
   → The contrast check is documented as a manual recommendation, not a
   completion gate.
2. **Why** is there no automated or gated check?  
   → The design spec was written assuming manual QA would cover it; the agent
   prompts do not require contrast verification before a UI component is
   marked done.
3. **Why** is manual QA insufficient?  
   → Manual checks are skipped under deadline pressure and do not cover all
   theme/surface combinations systematically.
4. **Why** don't agent prompts require context coverage?  
   → The UI designer and reviewer agents reference accessibility (WCAG) as a
   principle but do not make it a named step in the completion criteria.
5. **Root cause (process exposure):** UI agent prompts reference accessibility
   standards without mandating verification across all visual contexts as a
   condition of done. This is a gap waiting to become a defect on any project
   with dynamic backgrounds or multiple color schemes.

### Diff summary

Added "Visual Context Completeness" clause to `ui-designer.md` and "Visual
Context Coverage" item to `code-reviewer.md`. Both frame this as a verification
requirement, not a defect assertion, since the evidence shows a process gap
rather than a confirmed breakage.

---

## P6 — Cross-platform parity gap shipped silently

### Evidence

- `docs/IMPLEMENTATION_STATUS.md:282` — "Wear OS counterpart **not started**"
  appears only in a section header note.
- `docs/IMPLEMENTATION_STATUS.md:332` — Wear OS companion: "Not started. No
  Android Wear module exists." watchOS companion is 10/16 stories; Wear OS is 0
  stories started, 0 explicitly flagged as ⏳ in any planning commit.
- No commit or planning document at V1.5 inception flags the Wear OS asymmetry
  as a tracked gap; it surfaced only in the final status table, not in the
  feature plan.

### 5-Whys

1. **Why** was Wear OS not flagged during V1.5 planning?  
   → The V1.5 scope was framed around the watchOS companion; Wear OS was
   mentioned as a future item but not recorded as a gap in the status tracker.
2. **Why** wasn't the gap visible in the project status?  
   → The status table only lists what is being built; platforms not started
   were silently omitted, not recorded as ⏳.
3. **Why** is silent omission the default?  
   → Mobile developer agent prompts focus on the platform being implemented;
   they do not require recording the state of all other target platforms.
4. **Why** does this matter at release?  
   → Users on the omitted platform discover the gap at release, creating
   support burden and reputation risk that could have been managed earlier.
5. **Root cause:** No agent prompt requires that all target platforms be
   explicitly recorded in the status document when a feature ships on a
   subset of them.

### Diff summary

Added "Cross-Platform Parity" clause to `mobile-developer.md` and
"Cross-Platform Parity Coverage" principle to `qa-expert.md`. Both require
that unimplemented platforms be recorded as ⏳ explicitly, not omitted.

---

## P7 — Manual config step in docs, not enforced by CI

### Evidence

- `docs/IMPLEMENTATION_STATUS.md:99` — Watch complication: "Manual step
  required: add Widget Extension target in Xcode targeting watchOS, add
  App Group." Swift code complete; deployment blocked by IDE-only operation.
- `docs/IMPLEMENTATION_STATUS.md:556` — "CI step will warn (not fail) until
  the target is registered." Explicit acknowledgement that the gate is advisory.
- `.github/workflows/build-ios.yml` — App Group entitlement check added as
  a warning step, not a hard-fail gate.

### 5-Whys

1. **Why** is a feature blocked by a manual IDE step that CI does not enforce?  
   → The step was documented in `docs/release-guide.md` as something a human
   must do in an IDE; CI was set to warn rather than fail.
2. **Why** is a warning acceptable where a failure would catch the gap?  
   → The deployment engineer chose warn-not-fail to avoid blocking CI before
   the step was done; the intent was "don't break the build while we fix this."
3. **Why** does "warn to avoid breaking the build" become permanent?  
   → Once a warning lands, there is no automatic escalation path; warnings
   accumulate and stop being read.
4. **Why** wasn't an override / exception process used instead?  
   → No agent prompt distinguishes between "this step cannot be automated, use
   an override" and "downgrade the gate to a warning."
5. **Root cause:** Deployment agent prompts do not prohibit advisory-only
   gates and do not require a documented override process when a prerequisite
   genuinely cannot be automated.

### Diff summary

Added "No Advisory-Only Gates" principle to `deployment-engineer.md` and
"CI Gate Completeness" principle to `test-automator.md`. Both require hard-fail
gates; if automation is impossible, an explicit documented override process is
required rather than a permanent warning.

---

## P8 — Tests green, feature broken end-to-end

### Evidence

- Commit `7b6ba95` — "green pipeline on iPhone 17 + Android — 52/52 each"
  (all integration tests passing).
- Commit `415eb2c` (later) — "mantra everywhere it matters": streak SQL and
  History reads fixed; these were broken while the suite reported green because
  no test crossed from the dual-counter write to the streak/History read.
- Commit `020a216` — integration regression test added for the dual-counter
  divergence after the bug was found. The 21 unit tests in `2fa2869` covered
  the counter surface in isolation; none traversed the real DB path through
  to the History or streak surfaces.
- Commit `aab7847` — streak widget test: `currentStreak()` was not stubbed in
  the mock, causing the test to pass against an incomplete mock — not real
  behavior.

### 5-Whys

1. **Why** did the test suite report green while a cross-surface path was broken?  
   → Unit tests mocked the database; the mock did not reflect the actual query
   behaviour for the new counter column.
2. **Why** were mocked unit tests the only coverage for the cross-surface path?  
   → The feature's test plan covered the originating surface (counter) but did
   not identify History and streak as downstream consumers requiring their own
   integration tests.
3. **Why** is downstream surface coverage not part of the test plan?  
   → Test automator and QA prompts require integration tests but do not
   specifically require that each downstream surface of a new entity be covered
   by a test that crosses the real data boundary.
4. **Why** does this distinction matter?  
   → Mocked tests verify the contract between the test and the mock, not the
   contract between the application and its real data layer. A feature that
   crosses surfaces only reveals its breakage on the cross-boundary path.
5. **Root cause:** Test agent prompts require integration tests but do not
   mandate that cross-surface paths — from write surface through to each read
   surface — be exercised against real dependencies.

### Diff summary

Added "Cross-Surface Integration Coverage" principle to `test-automator.md`
and `qa-expert.md`. The clause specifically requires at least one integration
test per new entity that traverses the full path through real dependencies,
not mocks, from origin surface to each downstream consumer.

---

## P9 — Reviewer did not catch P1–P8

### Evidence

- PR `76ac3a7` — Merged branch included commit `da3243e` (sync UI with no
  transport, P1). PR title was "fix: onboarding Get Started button." Review
  did not flag the UI-without-transport pattern.
- No evidence in git log of any review comment catching P1–P8 patterns; all
  were caught by the author in reactive follow-up commits or doc patches.

### 5-Whys

1. **Why** did the reviewer not flag the UI stub?  
   → The reviewer checked functionality of the PR's stated purpose (onboarding
   button); the sync UI was in a different file and outside the stated scope.
2. **Why** was the wider change not reviewed?  
   → Code reviewer prompts list checklist items for security, quality, and
   performance but have no named item for "trace every UI control to an
   implementation."
3. **Why** don't the P1–P8 checks appear in the reviewer checklist?  
   → The checklist predates this RCA; it was not designed with the failure
   patterns observed here.
4. **Why** does a missing checklist item matter?  
   → Reviewers operate under time pressure; unnamed checks are systematically
   skipped, not forgotten.
5. **Root cause:** The code reviewer's checklist does not include P1–P8 as
   named, explicit items. Without named items, they are invisible to review.

### Diff summary

Added "Integration Surface Checklist" section to `code-reviewer.md` that
names P1–P8 explicitly as review items. Each pattern is actionable and
domain-generic.

---

## Future agent proposals

The following agent capability was identified as potentially valuable but is
**not shipped in this PR** to comply with the hard rule against new agents.

**Status Verification Agent** — An agent whose sole role is to walk every ✅
row in a project status document, locate the cited artifact, and confirm it
exists and is reachable. This would automate the P2 check. Proposed as a
future addition after the generic guardrails in the existing agents have
been validated across at least two projects.
