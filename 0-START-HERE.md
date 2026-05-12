# Composition Stack — Starter

This folder is the companion to the blog post **["Composition Stack for AI-Assisted .NET Development"](../composition-stack-ai-assisted-dotnet.md)**. It's a starter set of the six surfaces the post describes, written as if a real project — a fictional field-service app called **FieldKit** — were using them.

You're not meant to read this folder cover-to-cover. You're meant to:

1. Skim one file at a time.
2. Decide whether the *shape* of that file fits your project.
3. Steal the shape. Throw out FieldKit. Write your own content.

> The point isn't the exact filenames. The point is the structure — six surfaces, each answering a different question, each addressable on its own.

## The six surfaces

| Layer | File | Answers |
|---|---|---|
| **Foundation** | [`README.md`](./README.md) | What is this? Who is it for? How do I run it? |
| **Foundation** | [`CLAUDE.md`](./CLAUDE.md) | How do we do things here? What have we already decided? |
| **Wiring** | [`.mcp.json`](./.mcp.json) | What tools can the agent actually call? |
| **Wiring** | [`ux-flows.md`](./ux-flows.md) | What are the primary paths users take through the product? |
| **Design System** | [`design.md`](./design.md) | What does this product look like? |
| **Interactions** | [`interactions.md`](./interactions.md) | How does this product feel? |
| **Architecture** | [`architecture.md`](./architecture.md) | How is this product built? |
| **Plan** | [`plan.md`](./plan.md) | What are we building next, and what are we explicitly not building? |

Eight files. Six surfaces. The README + CLAUDE pair is the *Foundation* — most projects need both. The `.mcp.json` + `ux-flows.md` pair is the *Wiring* — one is for the agent, one is for everyone.

## How to use it

**Copy the folder.** Drop it into your project root (or a `docs/` subfolder — see the note in each file's header).

**Rename `FieldKit` to your product name.** A quick find-replace across the folder is fine. The example content is there to show shape, not to be kept.

**Delete what doesn't apply.** If you don't have animations yet, gut `interactions.md` to one sentence: *"No motion yet. Default to platform defaults until we decide otherwise."* That's still useful. A document that says "we haven't decided" is better than a missing one — the agent stops guessing.

**Keep them alive.** Stale beats missing only sometimes. A confidently wrong `architecture.md` is worse than no `architecture.md`. When you make a decision that contradicts a file, update the file in the same PR.

## What each file is shaped like

Each `.md` file in this folder follows the same structure:

```
<!-- ─── How to adapt ────────────────────────────────────────────────── -->
<!-- A short note at the top of the file explaining what this file       -->
<!-- is for, what to keep, what to throw out, and what's specific to     -->
<!-- the FieldKit example. Delete this comment block once you've adapted -->
<!-- the file to your own project.                                       -->
<!-- ─────────────────────────────────────────────────────────────────── -->

# <Title>

(content — written as if FieldKit were a real project)
```

The "How to adapt" block at the top of each file is the only meta-content. Everything below it is the kind of thing you'd actually write for a real product. That's deliberate — readers need to see what a *filled* brief looks like, not a list of blanks.

## Why these eight files and not seven, or twelve

The survey behind this starter looked at ~120 brief-style markdown files across ~19 sample Uno Platform projects. The eight files here are the ones that earned their place across all of them. The patterns that didn't make the cut:

- **Separate `vision.md` / `prd.md`** — the top of `README.md` carries this fine for small teams. Split them out when the product backlog needs its own life.
- **Separate `decisions.md` (ADR log)** — useful, but most projects fold it into `CLAUDE.md` as a "Decisions" section. Promote it to a standalone file once the section is more than 20 entries.
- **Separate `gotchas.md`** — folded into `CLAUDE.md` as a "Known platform traps" section. Same promotion rule.

If your project grows past the boundaries of any one file, split. The composition stack is meant to flex, not to be sacred.

## Credits

The shape of these files is synthesized from real Uno Platform sample projects I've been building. The cleanest patterns came from a handful — a contract-style CLAUDE.md from one, an ADR-style decision log from another, a "philosophy paragraph" up front from a third. None of those projects is FieldKit. FieldKit is a composite, the same way a textbook example is a composite.

Use what fits. Throw out what doesn't.
