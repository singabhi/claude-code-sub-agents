# Integration Surface / UI-Logic Pairing / Stub Declaration Contract

**Introduced by:** `audit/naamjap-rca-generic-hardening`  
**Applies to:** all agent prompts in this collection  

This document describes three interlocking conventions that the 2026-05 audit
introduced. Any future agent edit that touches feature delivery, status tracking,
or code review must remain consistent with these conventions.

---

## 1. The Integration Surface Rule

Every user-visible control that *implies* a feature — a toggle, button, action
menu item, or labelled UI element — must be backed by a real implementation
before it is marked complete.

**What counts as "backed":**
- A service call, SDK call, or network request that executes when the control
  is activated.
- A preference read site: if the control writes a preference, there must be
  code that reads that preference and alters application behaviour accordingly.

**What does not count:**
- The presence of UI scaffolding alone.
- Copy strings, translation keys, or configuration placeholders.
- A no-op stub function, unless explicitly declared (see §2).

This rule applies across all agent roles: developer, reviewer, QA, architect.

---

## 2. Stub Declaration

A phased or UI-first approach is acceptable when the full implementation is
not yet ready — but only if the stub is **declared explicitly**.

**Stub declaration requires:**

1. The status entry uses an unambiguous stub marker (e.g., `🟧 UI-only / stub`).
2. The missing component is named (e.g., "no network transport", "SDK not wired",
   "preference has no read site").
3. A follow-up task exists to complete the implementation.

An undeclared stub is treated as a complete feature by every downstream agent,
reviewer, and status reader. It will be shipped as such.

---

## 3. Cross-Surface Propagation

When a new data entity, field, or capability is introduced, the change is not
complete until every surface that displays or operates on that entity is updated.

**Surfaces to enumerate for any new entity:**
- List and detail views
- Charts and aggregation cards
- History and calendar views
- Search and filter paths
- Home-screen or companion-device widgets
- Sync and export paths

Each surface must be either updated in the same change set, or explicitly
recorded as a follow-up task with the feature marked partial — not complete.

---

## 4. Status Accuracy

A story or feature is `✅ complete` only when:
- Its implementation can be located at a cited file path, route, or endpoint.
- The implementation handles the data it claims to handle (read sites for
  preferences, real transports for sync controls, real SDK calls for
  integrations).
- All downstream surfaces of any new entity it introduces are updated.

A ✅ that cannot be verified against the codebase must be downgraded to ⏳
until the artifact is confirmed.

---

## How these conventions map to agent files

| Convention | Agents carrying the rule |
|---|---|
| UI-Implementation Pairing (§1) | `code-reviewer.md`, `mobile-developer.md`, `full-stack-developer.md`, `product-manager.md` |
| Preference Read Sites (§1) | `code-reviewer.md`, `full-stack-developer.md` |
| Stub Declaration (§2) | `product-manager.md`, `mobile-developer.md` |
| Cross-Surface Propagation (§3) | `code-reviewer.md`, `architect-review.md`, `full-stack-developer.md` |
| Status Accuracy (§4) | `documentation-expert.md`, `product-manager.md` |
| Visual Context Completeness | `ui-designer.md`, `code-reviewer.md` |
| Cross-Platform Parity | `mobile-developer.md`, `qa-expert.md` |
| CI Gate Completeness | `deployment-engineer.md`, `test-automator.md` |
| Cross-Surface Integration Tests | `test-automator.md`, `qa-expert.md` |
