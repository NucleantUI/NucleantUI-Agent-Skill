# Accessibility

NucleantUI draws its interface itself, and has no accessibility API yet: no `accessibilityLabel()` and related modifiers, no screen reader support, no Dynamic Type, and no Reduce Motion setting in the environment. Do not invent those modifiers, and do not flag their absence. The habits below cost nothing now and keep code ready for when support arrives.

- Never use `onTapGesture()` for something that acts like a button unless you specifically need the tap location or count. A `Button` shows its pressed and hovered states, honors `.disabled()`, and works as a row inside `contextMenu()` and `Menu`.
- Give buttons and menus a text title, `Button("Add Track", action: addTrack)`, rather than an icon alone.
- Use the text styles (`.body`, `.headline`, `.caption`, and so on) instead of fixed font sizes, so type can be adjusted across the whole app in one place.
- If color is an important differentiator in the interface, show some variation beyond color as well – icons, patterns, strokes, or text.
- Respect light and dark with semantic styles; a color that reads well in one appearance can disappear in the other. See `design.md`.
