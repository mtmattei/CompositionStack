<!-- ─── How to adapt ────────────────────────────────────────────────── -->
<!-- ux-flows.md describes the *primary user paths* through your product. -->
<!-- Not every screen. Not every edge case. The two or three flows that  -->
<!-- a new teammate (or agent) needs to picture to understand the shape   -->
<!-- of the product.                                                      -->
<!--                                                                      -->
<!-- Keep each flow to one screen of text. ASCII or numbered lists beat   -->
<!-- elaborate diagrams here — the file lives in source, not Figma.       -->
<!-- ─────────────────────────────────────────────────────────────────── -->

# FieldKit — UX Flows

The shape of the product, written as a small number of paths.

## The day-in-the-life flow

The flow the product is built around. Everything else exists to support this one.

```
Launch app
   │
   ▼
Sign in (PIN or biometric)
   │
   ▼
Today screen  ◄─────────────────────────────┐
   │                                          │
   ▼                                          │
Open job  ───────► Job detail                 │
                       │                      │
                       ▼                      │
                 Pick part(s)                 │
                       │                      │
                       ▼                      │
                Mark complete  ───────────────┤
                       │                      │
                       ▼                      │
                Sync queue                    │
                       │                      │
                       └──── next job  ───────┘
```

**Notes on the flow:**

- **Sign-in is fast.** PIN entry is 4 digits, biometric where available. We assume returning users every day; this is not a first-impression surface.
- **Today is the home.** Every loop ends back here. If a user can't get back to Today in one tap from anywhere, that's a bug.
- **Sync queue is invisible most of the time.** It's a chip in the chrome that shows "12 pending" when there's pending work, and disappears when the queue drains. It's not a screen the user navigates to deliberately.

## The first-run flow

The day-one flow. Runs once, ever, per device.

```
Launch app
   │
   ▼
Welcome screen (1 screen, 1 CTA)
   │
   ▼
Sign in with company credentials (OIDC)
   │
   ▼
"Allow location" (skippable, deferred-asks supported)
   │
   ▼
Set local PIN (4 digits, used after first sign-in)
   │
   ▼
Today screen — first-time empty state
```

**The first-time empty state on Today is important.** When the technician has no jobs assigned yet, we render a single line: *"You're all set. Jobs will appear here when dispatch assigns them."* Not a marketing screen. Not a tour. The dispatcher controls the timing of the user's first real interaction.

## The offline → online resync flow

The flow that's invisible when it works and catastrophic when it doesn't.

```
Network drops mid-shift
   │
   ▼
User keeps working
   │
   ▼
Every action queues locally (parts picked, completions, photos)
   │
   ▼
Sync queue chip shows "N pending" with a yellow dot
   │
   ▼
Network returns
   │
   ▼
Sync starts automatically. Chip animates while syncing.
   │
   ▼
On success: chip drains to zero, then disappears
   │
   ▼
On conflict (rare): job-detail surfaces a "Reconcile" banner
```

**Constraints:**

- The user never has to think about sync. No "tap to sync" button.
- Conflicts are surfaced where the user already is (on the job they touched), not as a global notification.
- Until Phase 3, conflict resolution is **last-write-wins** server-side. The reconcile banner is informational only.

## What's deliberately not in this file

- **Settings screens.** They have their own flow, but it's secondary. Document them where they're built.
- **Admin / dispatcher flows.** Out of scope for FieldKit-the-technician-app.
- **Error screens.** Treated as states inside the flows above (see [`interactions.md`](./interactions.md) for the state machines).
