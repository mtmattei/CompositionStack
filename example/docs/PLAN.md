<!-- ─── How to adapt ────────────────────────────────────────────────── -->
<!-- PLAN.md turns vague intent into something an agent (or a new        -->
<!-- contributor) can execute against. Phases, what's in each, what's    -->
<!-- explicitly out.                                                     -->
<!--                                                                     -->
<!-- The *don't-do list* is as important as the to-do list. It prevents  -->
<!-- the agent from helpfully scaffolding Phase 2 code into Phase 1       -->
<!-- files. Keep it.                                                     -->
<!-- ─────────────────────────────────────────────────────────────────── -->

# FieldKit — Plan

A scoped roadmap. Phase by phase, what we're shipping, what we're explicitly not, and how we'll know we're done.

## Current phase: Phase 1 — Read-only inventory

**Goal:** A technician can sign in, see today's jobs, open one, and view the truck inventory associated with it. No editing. No sync. The product compiles, runs on desktop + Android, and is usable end-to-end at the visual level.

**In scope:**

- Sign-in (OIDC via Uno.Extensions).
- Today screen — list of assigned jobs from the API, cached locally.
- Job detail — customer info + read-only parts list.
- Empty / loading / error states for both lists.
- Theme tokens applied throughout (no hardcoded values).
- Material theme, light + dark.

**Out of scope (do not scaffold):**

- Editing a parts list. Don't add `IsReadOnly="False"`. Don't surface a stepper, even disabled.
- Sync queue or `OutboxItem`. The DB schema for Phase 2 is fine to write down in [`ARCHITECTURE.md`](./ARCHITECTURE.md), but no code yet.
- Barcode scanning. No camera permission strings, no platform-specific scaffolding.
- Settings page. We don't have settings yet.
- Push notifications. The API supports them; we're not subscribing yet.

**Definition of done:**

- [ ] `dotnet build` clean on desktop + Android targets.
- [ ] All flows in [`UX-FLOWS.md`](./UX-FLOWS.md) (the "Day-in-the-life" up through "Open job") work on desktop with seeded data.
- [ ] Empty / loading / error states render and don't break the layout.
- [ ] No hex literals in XAML; all tokens come from `DESIGN-BRIEF.md`.
- [ ] PR includes a one-paragraph "how I tested" note from a human.

## Next: Phase 2 — Edit + sync

**Goal:** The technician can pick parts on a job and mark it complete. Everything queues locally and syncs when connectivity returns. The Phase 1 read-only views become writable.

**In scope:**

- Parts picker (`PartChip` interactions per [`INTERACTION-SPEC.md`](./INTERACTION-SPEC.md)).
- Mark complete + swipe interactions on `JobCard`.
- Sync queue (`OutboxItem` schema, drain loop, retry/backoff).
- `SyncBadge` in the chrome.
- Conflict surfacing (informational banner — no UI for resolution).

**Out of scope:**

- Barcode scanning.
- Reconcile UI beyond the informational banner.
- Multi-device handoff (the sync layer supports it; the UX doesn't).

**Definition of done:**

- [ ] The "offline → online resync" flow in [`UX-FLOWS.md`](./UX-FLOWS.md) works end-to-end with airplane-mode testing.
- [ ] Sync drains correctly after 100 queued items.
- [ ] No unhandled exceptions during network drop mid-action.

## Later: Phase 3 — Barcode and reconcile

**Goal:** Camera-driven part lookup and proper conflict resolution.

**In scope:**

- Camera permission flow (iOS + Android).
- Barcode scanner overlay invoked from the parts picker.
- Reconcile UI when the server rejected a write due to conflict.
- Conflict policy revisit (move off last-write-wins where it matters).

**Out of scope (still):**

- Admin / dispatcher views.
- Offline-first photo capture (separate plan, separate file when we get there).

## Don't-do list

Independent of phase. These come up; they're not on the roadmap.

- **A dashboard.** FieldKit is the technician app, not the management app. Dashboards live in a different product.
- **A "tour" or onboarding overlay.** The technician knows what a list is. The first-run flow in [`UX-FLOWS.md`](./UX-FLOWS.md) is the entire welcome surface.
- **A custom navigation framework.** Use Uno.Extensions regions. We've tried two homegrown alternatives in past projects; both were eventually replaced.
- **Cross-platform UI tests in Phase 1 or 2.** Cost > value at this scale. Desktop UI tests in Phase 2 are sufficient.
- **A second theme family** (e.g. Fluent in addition to Material). Material is the decision. Revisit only if a customer explicitly demands the Windows-native look — and weigh against the rewrite cost.
- **Inlining a third-party design system.** No FluentUI, no MUI, no whatever-the-design-team-saw-this-week. If a component needs to exist, it goes into our own `Controls/`.

## Decision rules (when work doesn't fit a phase cleanly)

If a piece of work straddles two phases or doesn't appear here:

1. Is it in the Don't-do list? **Stop.**
2. Does it have a brief covering it ([`ARCHITECTURE.md`](./ARCHITECTURE.md), [`DESIGN-BRIEF.md`](./DESIGN-BRIEF.md), [`INTERACTION-SPEC.md`](./INTERACTION-SPEC.md))? **Follow the brief.**
3. Is it a one-line change that unblocks the current phase? **Make it.** Note in the PR.
4. Is it bigger? **Stop and ask.** Update `PLAN.md` first, then write code.

The rule is the same for humans and agents: the plan is the contract. If the contract is wrong, change the contract before the code.

## Open questions

Things that need a decision but don't yet have one. Move to the `Decisions` table in [`CLAUDE.md`](../CLAUDE.md) when resolved.

- **Photo capture format.** Phase 3 needs photos attached to jobs. JPEG vs. HEIC vs. WebP — depends on what the API ingests cheaply. Owner: backend team.
- **Background sync on iOS battery saver.** iOS aggressively throttles. Worth investigating whether we surface the throttle to the user. Owner: not yet assigned.
- **Telemetry framework.** We have logging. We don't have metrics. App Insights vs. OpenTelemetry vs. just-logs-for-now. Defer until Phase 2 ships.
