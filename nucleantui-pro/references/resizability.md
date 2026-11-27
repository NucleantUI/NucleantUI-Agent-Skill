# Resizable windows and adaptive layouts

Treat window dimensions as changeable throughout a window's lifetime. Desktop windows on macOS, Linux, and Windows are resized freely, a browser tab running the WebAssembly build is resized with the page, phones rotate, and Android runs apps in split screen and free-form windows.

Do not introduce new layout code solely to modernize working code. Recommend changes only for requested behavior or a concrete layout problem.


## Choosing layouts

- Base layout choices on the space the view is offered, not on the platform or the device. Measure it with `onGeometryChange()` or `GeometryReader`, or let a `Layout` decide; never branch on the platform to choose a layout.
- Give screens that assume everything fits, such as setup flows and introductions, a `ScrollView`, so a short window cannot push their bottom actions out of reach.
- When different sizes select different view hierarchies, keep selections and navigation paths above the branch. State inside a replaced hierarchy loses its view identity and resets.
- `.relativeSize(width:height:)` sizes a view as a fraction of what its parent offers, which often replaces a `GeometryReader` that only computed a fraction.
- `visualEffect()` changes appearance from geometry without affecting layout, and is safe where layout must not change.


## Safe areas

- Remove padding that imitates a system safe-area inset, such as a fixed bottom gap meant to clear the home indicator or a status bar. NucleantUI already respects the window's safe area, and its size varies by device and by whether the keyboard or status bar is showing.
- When interactive or readable content must extend beyond the safe area, specify the intended edges, for example `.ignoresSafeArea(edges: .top)`. Unqualified `ignoresSafeArea()` is appropriate for decoration such as backgrounds, gradients, and images; on content it can put text and controls beneath system bars.
- `.background(style)` with a `ShapeStyle` already runs into the safe area at the edges the view touches, so a bar's or a screen's color reaches the window edge without `ignoresSafeArea()`.
- Use `safeAreaPadding()` only for intentional extra spacing inside the safe area, not as a guessed inset.


## Bars

- Prefer the `NavigationStack` bar and its `.bottomBar` and `.status` placements to custom bars layered in a `ZStack` with guessed padding.
- Hide the status bar on iOS and Android with `statusBarHidden()`; the safe area grows into its strip.
