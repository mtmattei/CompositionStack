<!-- ─── How to adapt ────────────────────────────────────────────────── -->
<!-- CLAUDE.md is the *how*. It's the conventions, rules, and decisions  -->
<!-- that aren't visible from reading the code. Think of it as the brief -->
<!-- you'd hand a new teammate so they don't have to ask three           -->
<!-- mid-sprint clarifying questions.                                    -->
<!--                                                                     -->
<!-- It is also the agent's session-start preamble. Everything below     -->
<!-- works for both audiences. Don't write a separate "AI prompt" file.  -->
<!--                                                                     -->
<!-- Sections that earn their place:                                     -->
<!--   - Read order (where to start)                                     -->
<!--   - Conventions (XAML, code, naming, anti-patterns)                 -->
<!--   - When to ask vs. decide (the contract)                           -->
<!--   - Verification checklist (what to run before claiming done)       -->
<!--   - Known platform traps (Uno/WinUI gotchas with mitigations)       -->
<!--   - Decisions (an ADR-lite log — promote to its own file if it      -->
<!--     grows past ~20 entries)                                         -->
<!-- ─────────────────────────────────────────────────────────────────── -->

# FieldKit — Conventions and Contract

Read this before you touch the code. Whether you're an engineer joining the team or an agent picking up a task, the same rules apply.

## Read order

When in doubt, read in this sequence:

1. [`README.md`](./README.md) — what FieldKit is.
2. This file (`CLAUDE.md`) — how we work.
3. [`architecture.md`](./architecture.md) — how the code is organized.
4. The one brief that matches the task — `design.md`, `interactions.md`, `ux-flows.md`, or `plan.md`.

Don't read everything at session start. Read what the task touches.

## Conventions

### XAML

- **Use `x:Bind`.** Use `{Binding}` only for `DataTemplate` scenarios where `x:Bind` doesn't work, and add a comment explaining why.
- **One root layout per page.** A page with three nested `Grid`s wrapping a `StackPanel` is almost always wrong.
- **Don't put logic in code-behind.** If a page has more than ~20 lines of `.xaml.cs`, push the logic into the MVUX model.
- **Reuse styles. Never inline a color, a font size, or a thickness** that exists in [`design.md`](./design.md). If the token doesn't exist yet, add it to `design.md` in the same PR.

### Code

- **MVUX is the default.** Use `IFeed<T>`, `IState<T>`, `IListFeed<T>` for app state. Use `FeedView` to bind. See [`architecture.md`](./architecture.md) for the decision rule.
- **`async` all the way down.** No `.Result`, no `.Wait()`, no `Task.Run` to escape sync-over-async — fix the call chain instead.
- **One responsibility per service.** `IInventoryService` reads/writes inventory. It does not also do telemetry or auth.
- **Don't catch exceptions you can't handle.** Let them bubble. The hosting layer logs them.

### Naming

- Pages: `<Thing>Page.xaml`. Models: `<Thing>Model.cs` (MVUX) or `<Thing>ViewModel.cs` (MVVM, exceptional).
- Services: `I<Thing>Service` / `<Thing>Service`. Records: `<Thing>` (no suffix).
- Async methods end in `Async`. Test methods end in `_Should<Outcome>`.

### Anti-patterns (we've already tried these)

- **`Dispatcher.RunAsync` from a service.** Services are platform-agnostic. If you need to marshal back to the UI thread, do it at the binding layer or use an MVUX feed.
- **Manual `INotifyPropertyChanged`.** MVUX states handle change notification. If you're reaching for `INotifyPropertyChanged`, you're in the wrong layer.
- **Region-less navigation hacks.** All navigation flows through Uno.Extensions regions. See [`architecture.md`](./architecture.md).
- **A new color "just for this one screen."** That color always ends up everywhere. Add to `design.md` or use what exists.

## When to ask vs. when to decide

The contract:

| Situation | Decide | Ask |
|---|---|---|
| Token, style, or layout choice covered in [`design.md`](./design.md) | ✅ | |
| New token, new style, or visual deviation | | ✅ |
| Adding a service / DI registration | ✅ | |
| Adding a new external dependency | | ✅ |
| Animation timing within ranges in [`interactions.md`](./interactions.md) | ✅ | |
| New animation pattern | | ✅ |
| Phase 1 work as scoped in [`plan.md`](./plan.md) | ✅ | |
| Anything labeled "Phase 2/3" or "Non-goals" | | ✅ |
| Refactor across more than three files | | ✅ |

**Default:** if it's covered by a brief, follow the brief. If you're about to write something that contradicts a brief, stop and either update the brief (with rationale) or ask.

## Verification checklist

Before claiming a task is done:

- [ ] `dotnet build` passes for `net10.0-desktop` with zero warnings introduced.
- [ ] If UI changed, ran the desktop target and exercised the changed path manually. Screenshot in PR.
- [ ] If a brief changed, the change is in the same PR as the code.
- [ ] No new file in `Platforms/` without a one-line comment explaining the platform-specific reason.
- [ ] No XAML resource literal that should be a token.
- [ ] `dotnet format` is clean.

This is the floor, not the ceiling. Type-check and tests verify *code correctness*, not *feature correctness*. If you can't actually use the feature, say so in the PR.

## Known platform traps

The traps below have bitten us before. Fix them at the source if you hit them again — don't paper over.

- **MVUX `IFeed` returning `null` from a service that swallowed an exception.** Don't swallow. Let the feed surface the error and let `FeedView` render it.
- **`x:Bind` to a method that throws on null.** `x:Bind` will silently render blank. Add a null check or use a converter.
- **Android keyboard pushes the content up, iOS overlays it.** Use Uno Toolkit's `SafeArea` and test both before declaring a form done.
- **`SkiaSharp` preview versions vs. Uno.Sdk stable.** If you bump SkiaSharp to a preview, use the matching Uno.Sdk dev version, and set `<UnoDisableLottieSkiaVersionCheck>true</UnoDisableLottieSkiaVersionCheck>`.
- **Region navigation through a `ContentControl` without `Region.Attached`.** The control needs both `Region.Attached="True"` and to be reachable from a parent `Region.Navigator`. A missing `Region.Attached` produces a blank page with no error.
- **Material theme keys after a version bump.** Bumping `Uno.Material` sometimes renames resource keys. If a page goes blank after an upgrade, suspect the theme keys first.

## Decisions

A short log of choices that future-us would otherwise have to re-litigate. One entry per row. Add to the bottom; never edit history.

| Date | Decision | Why | Risk |
|---|---|---|---|
| 2026-04-22 | MVUX over MVVM for FieldKit | Async data flows dominate (sync queue, paged inventory). MVUX feeds remove `INotifyPropertyChanged` boilerplate. | Smaller community than CommunityToolkit.Mvvm. Mitigated by keeping MVVM available for one-off pages. |
| 2026-04-22 | Uno.Extensions navigation regions, not raw `Frame` | Regions compose, support DI, and survive deeplinks. Same shell pattern across desktop/mobile. | Region attachment is a known footgun (see Known platform traps). |
| 2026-04-30 | Material theme (MD3), not Fluent | Higher contrast tokens, better default touch sizing for the field-tech use case. | Visual mismatch with native Windows shell. Acceptable — utility app, not a Windows-first app. |
| 2026-05-05 | Offline-first via local SQLite + sync queue | The user works in a basement or driveway. Connectivity is the exception. | Conflict resolution complexity. Constrained to "last write wins" until Phase 3. |
| 2026-05-09 | Defer barcode scanning to Phase 3 | The platform-specific work (camera permissions, MAUI Essentials replacement) is half the cost of Phases 1+2 combined. Ship a working sync loop first. | None — explicitly scoped in `plan.md`. |

When this table passes ~20 rows, promote it to a standalone `decisions.md` and link from here.
