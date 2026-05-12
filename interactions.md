<!-- ─── How to adapt ────────────────────────────────────────────────── -->
<!-- interactions.md describes how the product *feels*: animations,      -->
<!-- transitions, state transitions, edge cases (loading / empty /        -->
<!-- error). Visual rules live in design.md. This file is about motion    -->
<!-- and time.                                                            -->
<!--                                                                      -->
<!-- Make timing tokens, not magic numbers. Make state machines, not      -->
<!-- prose. Make loading / empty / error first-class, not afterthoughts.  -->
<!-- ─────────────────────────────────────────────────────────────────── -->

# FieldKit — Interactions Brief

## Motion philosophy

Motion in FieldKit communicates **state change**, never decoration. If a movement doesn't tell the user "something is now different," remove it. We err toward fast and subtle — the user has work to do, and a 600ms transition feels like a stall when you've tapped the same button two hundred times today.

When in doubt: shorter, softer, fewer.

## Timing tokens

Defined in `Themes/Motion.xaml`.

| Token | Duration | Easing | Used for |
|---|---|---|---|
| `Motion.Snap` | 120ms | `CubicBezier(0.4, 0.0, 0.2, 1)` | Button press feedback, chip toggle |
| `Motion.Quick` | 200ms | `CubicBezier(0.4, 0.0, 0.2, 1)` | Page transitions, list item enter/exit |
| `Motion.Standard` | 280ms | `CubicBezier(0.4, 0.0, 0.2, 1)` | Bottom sheet, drawer |
| `Motion.Slow` | 480ms | `CubicBezier(0.2, 0.0, 0.0, 1)` | First-run welcome (one place only) |

**Rules:**

- Skeletons appear at 200ms, not sooner. Anything faster looks like a flash and worsens the perception of speed.
- Page transitions are `Motion.Quick`. We're a utility app — no `Motion.Slow` on navigation.
- Never animate the same property twice in a single user action (e.g., fade + slide). Pick one.

## State models

Every visible component has the same four primary states. Components that read or sync also have loading and error.

```
        idle ────tap────► active ────release────► idle
          │                   │
          └──disabled         └──long-press─────► confirming
```

For data-bound components:

```
        loading ────success────► ready
            │
            └──fail──► error ────retry────► loading
```

**Use MVUX feeds for these.** A `FeedView` renders the right template for each state. We don't write loading/error logic in code-behind.

## Component interactions

### `JobCard`

| Trigger | Behavior | Timing |
|---|---|---|
| Tap | Background tints to `Md.Sys.Color.Primary` at 8% opacity, then navigates | `Motion.Snap` ripple, `Motion.Quick` page transition |
| Long-press | Haptic tick (Android/iOS). Reveals "Set status" sheet from bottom. | `Motion.Standard` |
| Swipe right | Marks complete inline. Card slides off-screen left, list closes gap. | `Motion.Quick` slide, `Motion.Quick` gap close |
| Swipe left | Returns the card from "completed" to "active". Inverse of above. | Same as above |

### `PartChip` <a name="partchip"></a>

The chip behavior is tactile by intent. The technician's hands are full; the affordance has to feel deliberate.

| Trigger | Behavior | Timing |
|---|---|---|
| Tap | Quantity increments by 1. Number flips up (counter-style). Haptic tick. | `Motion.Snap` |
| Long-press (300ms) | Enters "decrement mode": chip background tints `Md.Sys.Color.Tertiary`. Next tap decrements. Mode auto-exits in 3s. | `Motion.Quick` tint, 3s timeout |
| Tap with shift (desktop) | Decrement by 1. Desktop affordance only. | `Motion.Snap` |

### `SyncBadge`

States: hidden → pending → syncing → success-flash → hidden.

| State | Visual | Transition in / out |
|---|---|---|
| Hidden | Component is collapsed, takes no space. | — |
| Pending | Yellow dot + count, e.g. "12". | Fade-in `Motion.Quick` |
| Syncing | Yellow dot replaced by a spinner. Count stays. | Cross-fade `Motion.Snap` |
| Success-flash | Green check, count "0", visible 1.2s. | Cross-fade `Motion.Quick` → auto-hide |
| Error | Red dot, count of failed items. Persists until tapped. | Fade-in `Motion.Quick` |

The component never animates while syncing if the device is low-power-mode (we read the platform flag and short-circuit to a static spinner glyph).

### `EmptyState`

No animation on first mount. Static. Animating an empty state suggests we're loading; we're not.

### `ErrorState`

The error illustration breathes — a 4s, 2% scale loop. This is the **only** ambient motion in the product. It exists so the user notices the screen at all when an error happens mid-shift.

## Page transitions

| From → To | Transition | Timing |
|---|---|---|
| Today → Job detail | Push from right. Job card stays visible until 50% (shared-element-style, but lightweight — no element ID matching). | `Motion.Quick` |
| Job detail → Today | Slide back to right. The card returns to its position in the list. | `Motion.Quick` |
| Anywhere → Sign-in (auth expiry) | Hard cut, no animation. Auth boundary should feel discrete. | 0ms |
| First-run flow | Step-by-step, each step fades through black for 280ms. Only place we do this. | `Motion.Standard` |

## Loading, empty, and error patterns

These are component-level, not page-level, except where the page itself is the unit of work.

| State | Pattern |
|---|---|
| Loading (component) | Skeleton block sized to the expected content. Appears at 200ms. Never spinner-on-page. |
| Loading (full-page) | Header renders immediately. Body is skeleton until ready. |
| Empty | `EmptyState` component (see [design.md](./design.md#components)). One line, no CTA except for the first-run case. |
| Error (component) | Inline message inside the component, with retry affordance. Don't bounce the user to a separate error page. |
| Error (full-page) | Reserved for auth, network unreachable on cold start, and unhandled exceptions. Includes the "breathing" illustration. |

## Edge cases worth calling out

- **Triple-tap on a chip.** Increments by 1, three times. We don't debounce because the technician sometimes legitimately picks 3 of the same part fast.
- **Backgrounded mid-sync.** Sync continues in a foreground service (Android) / background task (iOS). The badge resumes correct state on app return — no manual refresh.
- **Network drop mid-tap.** The tap completes locally (queues to sync). The sync badge updates. No error dialog. The user shouldn't see a network failure mid-action; that's a sync-time problem.
- **Multiple devices, same user.** A job marked complete on one device animates to "completed" on the other when it reconnects. The animation timing is the same as a manual swipe — the user shouldn't be able to tell the difference between local and remote completion.
