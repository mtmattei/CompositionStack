# Composition Stack

A starter set of context files for AI-assisted .NET / Uno Platform development.

This repo is the companion to the blog post **"Composition Stack for AI-Assisted .NET Development."** It's not an application. There's no code to run. It's a set of markdown files that show what the six "surfaces" of an AI composition stack look like when they're filled in — so you can lift the *shape* and write your own.

> The point isn't the exact filenames. The point is the structure — six surfaces, each answering a different question, each addressable on its own.

## What's in this repo

```
CompositionStack/
  README.md          ← you are here
  example/           ← a worked example: FieldKit (a fictional field-service app)
    README.md
    CLAUDE.md
    .mcp.json
    ux-flows.md
    design.md
    interactions.md
    architecture.md
    plan.md
```

The `example/` folder contains all eight files filled in as if a fictional Uno Platform app called **FieldKit** were using them. FieldKit isn't a real product. It exists only to give the templates something concrete to talk about — colors, animations, phases, decisions — so readers see what a *filled* brief looks like, not a list of blanks.

## The six surfaces

| Layer | File | Answers |
|---|---|---|
| **Foundation** | [`README.md`](./example/README.md) | What is this? Who is it for? How do I run it? |
| **Foundation** | [`CLAUDE.md`](./example/CLAUDE.md) | How do we do things here? What have we already decided? |
| **Wiring** | [`.mcp.json`](./example/.mcp.json) | What tools can the agent actually call? |
| **Wiring** | [`ux-flows.md`](./example/ux-flows.md) | What are the primary paths users take through the product? |
| **Design System** | [`design.md`](./example/design.md) | What does the product look like? |
| **Interactions** | [`interactions.md`](./example/interactions.md) | How does the product feel? |
| **Architecture** | [`architecture.md`](./example/architecture.md) | How is the product built? |
| **Plan** | [`plan.md`](./example/plan.md) | What are we building next, and what are we explicitly *not* building? |

Eight files, six surfaces. The README + CLAUDE pair is the *Foundation* — most projects need both. The `.mcp.json` + `ux-flows.md` pair is the *Wiring* — one is for the agent, one is for everyone.

## How to use it

1. **Read [`example/`](./example/) once.** Get a feel for what each file is shaped like and how they cross-reference each other.
2. **Copy the eight files into your own project root** (or a `docs/` subfolder). Drop them anywhere your agent looks at session start.
3. **Find-replace `FieldKit` to your product name.** The example content is there to show shape, not to be kept.
4. **Rewrite each file with your project's truth.** A doc that says "we haven't decided yet" is better than a missing one — the agent stops guessing.
5. **Keep them alive.** A confidently wrong `architecture.md` is worse than no `architecture.md`. When a decision changes, update the file in the same PR.

## What each file is shaped like

Every file in `example/` follows the same structure:

```
<!-- ─── How to adapt ────────────────────────────────────────────────── -->
<!-- A short note at the top explaining what this file is for, what to   -->
<!-- keep, what to throw out, and what's specific to the FieldKit         -->
<!-- example. Delete this comment block once you've adapted the file to   -->
<!-- your own project.                                                    -->
<!-- ─────────────────────────────────────────────────────────────────── -->

# <Title>

(content — written as if FieldKit were a real project)
```

The "How to adapt" block is the only meta-content. Everything below it is the kind of thing you'd actually write for a real product. That's deliberate — readers need to see what a *filled* brief looks like, not a list of blanks.

## Why these eight files and not seven, or twelve

The shape behind this starter is synthesized from ~120 brief-style markdown files across ~19 real Uno Platform sample projects. The eight files here are the ones that earned their place across the corpus. The patterns that didn't make the cut:

- **Separate `vision.md` / `prd.md`** — the top of `README.md` carries this fine for small teams. Split them out when the product backlog needs its own life.
- **Separate `decisions.md` (ADR log)** — useful, but most projects fold it into `CLAUDE.md` as a "Decisions" section. Promote it to a standalone file once the section is more than 20 entries.
- **Separate `gotchas.md`** — folded into `CLAUDE.md` as a "Known platform traps" section. Same promotion rule.

If your project grows past the boundaries of any one file, split. The composition stack is meant to flex, not to be sacred.

## A note on the example

FieldKit, the fictional app the example is built around, is a composite. It borrows the shape of a few real projects without being any one of them. The patterns embedded in the files are real — a contract-style CLAUDE.md, an ADR-lite decisions table, a philosophy paragraph at the top of every brief — but the product isn't.

Use what fits. Throw out what doesn't.
