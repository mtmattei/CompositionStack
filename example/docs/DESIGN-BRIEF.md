<!-- ─── How to adapt ────────────────────────────────────────────────── -->
<!-- DESIGN-BRIEF.md is the visual brief. Tokens, palette, typography,   -->
<!-- spacing, components. The constraints that make a hundred screens    -->
<!-- feel like one product.                                              -->
<!--                                                                     -->
<!-- Start with a one-paragraph aesthetic philosophy — the *why* behind  -->
<!-- the values. Without it, every token below looks arbitrary.          -->
<!--                                                                     -->
<!-- Tables beat prose for tokens. Resource keys beat hex values for     -->
<!-- usage. Examples beat descriptions for components.                   -->
<!-- ─────────────────────────────────────────────────────────────────── -->

# FieldKit — Design Brief

## Aesthetic philosophy

FieldKit is a tool, not an experience. The user is wearing gloves, standing in a basement, and resents anything that takes more than one tap. The design language is dense, high-contrast, and utility-first — closer to a flight-deck console than a consumer app. Color earns its place by meaning something. Animation earns its place by communicating state, not delight. White space is for legibility, never for atmosphere.

If a screen could be mistaken for a marketing page, redesign it.

## Color tokens

Defined as Material Design 3 tokens in `Themes/Light.xaml` and `Themes/Dark.xaml`. Reference by key. **Never hardcode hex.**

| Role | Key | Light | Dark | Used for |
|---|---|---|---|---|
| Primary | `Md.Sys.Color.Primary` | `#1F4D7A` | `#9CC9F2` | Primary actions, active nav, focus rings |
| OnPrimary | `Md.Sys.Color.OnPrimary` | `#FFFFFF` | `#0A2540` | Text/icons on Primary |
| Surface | `Md.Sys.Color.Surface` | `#F7F8FA` | `#10141A` | Page background |
| SurfaceContainer | `Md.Sys.Color.SurfaceContainer` | `#EAEDF2` | `#1B2330` | Cards, list rows |
| OnSurface | `Md.Sys.Color.OnSurface` | `#10141A` | `#E6EAF2` | Body text |
| OnSurfaceVariant | `Md.Sys.Color.OnSurfaceVariant` | `#48515E` | `#A8B0BD` | Secondary text, metadata |
| Outline | `Md.Sys.Color.Outline` | `#C5CAD3` | `#3D4654` | Dividers, disabled borders |
| Success | `Md.Sys.Color.Tertiary` | `#1F6F47` | `#7BD3A6` | Completed jobs, sync success |
| Warning | `Md.Sys.Color.SecondaryContainer` | `#C26A18` | `#F2B070` | Pending sync, near-empty stock |
| Error | `Md.Sys.Color.Error` | `#B3261E` | `#F2B8B5` | Sync failure, invalid input |

**Rules:**

- Two color decisions per screen, max. Background + accent.
- Never use Error or Warning as a decorative color. They mean what they say.
- Disabled state is `OnSurface` at 38% opacity. Don't introduce a new "muted" token.

## Typography

Single family: **Inter** (system fallback: `-apple-system, Segoe UI, Roboto`).

| Token | Size | Weight | Line height | Used for |
|---|---|---|---|---|
| `Md.Sys.Type.DisplaySmall` | 28 | 600 | 36 | Today screen header |
| `Md.Sys.Type.HeadlineMedium` | 22 | 600 | 28 | Page titles |
| `Md.Sys.Type.TitleLarge` | 18 | 600 | 24 | Job cards, section headers |
| `Md.Sys.Type.BodyLarge` | 16 | 400 | 24 | Body text |
| `Md.Sys.Type.BodyMedium` | 14 | 400 | 20 | Metadata, secondary text |
| `Md.Sys.Type.LabelLarge` | 14 | 600 | 20 | Button labels |
| `Md.Sys.Type.LabelMedium` | 12 | 500 | 16 | Chip text, badges |

**Rules:**

- Two type sizes per row, max. One for the thing, one for its metadata.
- Bold for hierarchy, never italic for emphasis. We render at small sizes on outdoor screens; italic loses too much.
- Numerals are tabular (`FontVariantNumeric="Tabular"`) on anything that lists quantities, prices, or counts. Misaligned numbers in a parts list looks broken.

## Spacing

8-unit base grid. Named tokens only — no raw thicknesses in XAML.

| Token | Value | Used for |
|---|---|---|
| `Spacing.XSmall` | 4 | Icon-to-text within a chip |
| `Spacing.Small` | 8 | Inside chip / badge, item gap in dense lists |
| `Spacing.Medium` | 16 | Default card padding, default row gap |
| `Spacing.Large` | 24 | Section gap |
| `Spacing.XLarge` | 32 | Top of page after the app bar |

Touch targets: **48dp minimum.** This is the only non-negotiable. Gloves.

## Elevation

We render almost flat. Three steps total.

| Token | Used for |
|---|---|
| `Elevation.0` | Page surface, non-interactive content |
| `Elevation.1` | Cards the user can act on (job cards, inventory rows) |
| `Elevation.3` | Bottom sheets, dialogs |

If you reach for Elevation.2 or 4-5, you're inventing. Stop.

## Components

### Job card

The unit of the Today screen. One card per assigned job.

- **Container.** `SurfaceContainer`, Elevation.1, 12dp corner radius, 16dp padding.
- **Title** (`TitleLarge`, `OnSurface`) — customer name.
- **Metadata** (`BodyMedium`, `OnSurfaceVariant`) — address, scheduled window.
- **Status chip** (top-right) — uses one of: Primary (active), Tertiary (completed), SecondaryContainer (queued).
- **Tap → full card** transitions to job detail. No secondary actions on the card.

### Inventory row

Used inside job detail. One row per part on the truck.

- **Two columns.** Left: part name + part number. Right: quantity stepper.
- **Height: 56dp.** Inside a `ListView` with `ItemContainerStyle` setting min height to ensure the touch target rule holds.
- **Quantity stepper** is two 48×48 buttons with a tabular numeral between them.
- **Color cue.** Background tints to `SecondaryContainer` at 12% opacity when stock is near zero.

### App bar

One per page. Title left, sync chip right, no overflow menu. If a page needs an action, it lives at the bottom (one button, full-width, primary).

## Iconography

Material Symbols (Rounded fill), 24dp by default, 20dp inside dense rows.

Two rules:

- **No icon without a label** unless the icon is universally understood (close, back, search).
- **No more than five distinct icons per screen.** If you need more, you're rendering a settings page or you've overloaded the screen.

## Accessibility floor

- Contrast ratio ≥ 4.5:1 for body text, ≥ 3:1 for large text and UI elements.
- Every interactive element has an `AutomationProperties.Name`.
- All flows are completable with screen reader.
- No information conveyed by color alone — pair with text or shape.

When you add a new component, run the WinUI contrast analyzer once before you call it done.

## What's not in this file

- **Motion and timing.** See [`INTERACTION-SPEC.md`](./INTERACTION-SPEC.md).
- **Layout rules per page.** Components compose. If a page needs a new layout rule, that's a sign it should be expressed as a new component instead.
- **Brand voice and copy tone.** Lives in the README's "Who it's for" section for now. Promote to a `voice.md` if FieldKit ever ships marketing copy.
