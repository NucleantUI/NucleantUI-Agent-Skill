# Design

## Creating a uniform design in this app

Prefer to place standard fonts, sizes, colors, stack spacing, padding, rounding, animation timings, and more into one shared table of constants, such as a `Theme` of static values, so they can be used by all views. This allows the app's design to feel uniform and consistent, and be adjusted easily.


## Stable, flexible layouts

- Text whose content changes at runtime – a counter, a value readout, a status line, a timer – gets a fixed frame sized for its longest value, such as `.frame(width: 64, alignment: .trailing)`, and a monospaced font where digits change. Otherwise its changing length resizes it and pushes the views beside it. Never leave such a text at its natural width in an `HStack` with a `Spacer` or flexible siblings.
- Keep sensible proportions. Do not let one element, such as a waveform or a scope, become a full-width sliver because other controls were laid out in one long row; group controls so displays keep a sensible aspect.
- Prefer flexible frames unless a fixed size can comfortably hold the content. Account for narrower windows and larger text.
- Preserve access to the same features at every window size. Additional width can show more of an existing hierarchy, such as a list and its detail together, without changing what the app can do.
- The minimum acceptable tap area on touch hosts is 44x44 points. Ensure this is strictly enforced; `.contentShape()` can enlarge a hit area without changing the drawing.
- Interfaces run under a mouse and under a finger. A finger never hovers, so `onHover()` can add emphasis but must never be the only way to reveal a control or information. A right click and a press held still both open a `contextMenu()`; keep its actions reachable elsewhere too.


## Standard styling

- Prefer hierarchical styles (`.primary`, `.secondary`, `.tertiary`) over manual opacity, so the style adapts to light and dark automatically.
- Use semantic surfaces – `.background`, `.secondaryBackground`, `.tertiaryBackground`, `.separator`, `.fill` – and `Color.dynamic(light:dark:)` for colors of your own, so the app follows the system appearance. A fixed `Color(hex:)` is the same in both appearances; use one for a surface only when the design is deliberately one appearance.
- When using `Form`, wrap controls such as `Slider` in `LabeledContent` so the title and control are laid out correctly. `LabeledContent` also works outside `Form` for any title-value display.
- Icons are SVGs shown with `SvgView`; use `.renderingMode(.template)` so an icon paints in the current foreground style and follows the appearance.


## Ensuring designs work for everyone

- Use `bold()` instead of `fontWeight(.bold)`.
- Only use `fontWeight()` for weights other than bold when there's an important reason – scattering around `fontWeight(.medium)` or `fontWeight(.semibold)` is counterproductive.
- Prefer the framework's default padding and stack spacing, or the app's theme constants, to scattered hard-coded values.
- Never use platform colors such as `UIColor` or `NSColor`; use `Color`.
- `.caption` is the smallest text style, at 12 points; use it carefully.
