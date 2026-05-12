> **This is an example, not a real product.**
>
> FieldKit doesn't exist. It's a fictional Uno Platform app invented for the [Composition Stack starter](../README.md) to give the eight context files something concrete to talk about. The `dotnet` commands won't build anything — there's no code in this repo. What you're reading is what *FieldKit's* `README.md` would look like *if* it were real. Read it for shape; throw it out when you copy the files into your own project.

<!-- ─── How to adapt ────────────────────────────────────────────────── -->
<!-- The README is the *what*. It answers three questions:               -->
<!--   1. What is this product?                                           -->
<!--   2. Who is it for?                                                  -->
<!--   3. How do I run it?                                                -->
<!-- Keep it short. Anything longer than two screens belongs in another   -->
<!-- file. If you find yourself writing about *how things are done*, stop -->
<!-- and put it in CLAUDE.md instead — that's the *how*.                  -->
<!-- ─────────────────────────────────────────────────────────────────── -->

# FieldKit

A cross-platform inventory app for field service technicians. Built with .NET 10 and Uno Platform. Targets Windows desktop, iOS, and Android from a single codebase.

## Who it's for

Field service teams — HVAC, electrical, plumbing — who arrive at a job site, pull parts from a van inventory, and need a record of what they used before they leave the driveway. The day-one user wears gloves, works in cold/wet conditions, and resents typing. The product is built around that user.

## What it does (today)

- **Today screen.** The technician's assigned jobs for the current shift, sorted by route.
- **Job detail.** Customer info, work order, and the parts available on the truck.
- **Inventory pick.** Tap or scan a part to add it to the job. Tap again to return it.
- **Sync queue.** Everything queues locally. Sync runs in the background when connectivity returns.

What it doesn't do yet: see [`plan.md`](./plan.md).

## Stack

- **.NET 10 / C# 14** — application code.
- **Uno Platform (latest stable Uno.Sdk)** — cross-platform UI from one codebase.
- **MVUX** — application state and async data flow. (For why not MVVM, see [`architecture.md`](./architecture.md).)
- **Uno.Extensions** — hosting, DI, navigation, authentication, HTTP, configuration, logging.
- **Uno Toolkit + Material** — controls and theming.

## Run it

Prereqs: latest stable .NET SDK and the Uno Platform workloads.

```bash
dotnet workload restore
dotnet build
```

To launch a target:

```bash
# Windows desktop (default for day-to-day work)
dotnet run --project FieldKit/FieldKit.csproj -f net10.0-desktop

# iOS simulator
dotnet build -t:Run -f net10.0-ios

# Android emulator
dotnet build -t:Run -f net10.0-android
```

## Project layout

```
FieldKit/
  FieldKit.csproj           single project, multi-target
  App.xaml(.cs)             host setup, DI registration
  Presentation/             pages and controls (XAML + .cs)
  Models/                   MVUX records and feeds
  Services/                 platform-agnostic services
  Platforms/                platform-specific code (#if ANDROID, etc.)
  Assets/                   images, fonts, app icons
docs/
  CLAUDE.md                 conventions, decision rules
  architecture.md           how the code is organized
  design.md                 visual language
  interactions.md           motion + state rules
  ux-flows.md               primary user paths
  plan.md                   roadmap + non-goals
.mcp.json                   agent tool registry
```

## Status

**Phase 1 in progress.** Read-only inventory view on Today/Job detail. Sync, edit, and barcode scanning are scoped but not built. See [`plan.md`](./plan.md) for the current cut line.

## Contributing

Read [`CLAUDE.md`](./CLAUDE.md) before opening a PR. It captures the conventions a new teammate (human or agent) needs in week one. The TL;DR:

- Prefer `x:Bind` over `{Binding}`.
- Default to MVUX. Use MVVM only for sample/throwaway pages, and say why in the PR.
- Touch targets are ≥ 48dp. The user is wearing gloves.
- Don't introduce a new color, font, or animation timing without updating the relevant brief.

## License

Example/template project. Adapt freely.
