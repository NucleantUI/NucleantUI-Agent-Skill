# Working on NucleantUI, or a library that extends it

Load this file only for NucleantUI's own sources, or a package that adds views, modifiers, or capabilities on top of NucleantUI. Everything else in this skill still applies.


## Following SwiftUI

- SwiftUI's API is the reference. Names, signatures, modifiers, and behavior should match SwiftUI's as closely as practical, weighed against how NucleantApplication and NucleantVulkan actually work. Where matching SwiftUI exactly fights the app or render model, work with what they do.
- NucleantUI's deliberate differences take precedence over SwiftUI: `@View`, `SystemTray`, and the additions listed in `api.md`. Never propose renaming them back.
- Implement every SwiftUI API – each type, modifier, and overload with the same name, such as `.task(priority:_:)` and `.task(id:priority:_:)` – as its own thing, from what its own documentation says it does. Never route one through another with a dummy argument, never decide which is "the base" that another builds on, and never make two API types share one implementation, such as `HStack` and `HStackLayout`. Do not recommend merging them, not even as an option.


## Extending from outside

- A library built on NucleantUI adds what it needs through Swift extensions and NucleantUI's public API, from its own package. Do not open with "change NucleantUI"; if a public hook is genuinely missing, say exactly which one and why.
- New render nodes are written in NucleantUI, or in the extending package, conforming to `VulkanRenderNode`; NucleantVulkan is not bent to fit one. Keep engine, device, and node APIs generic over their node types rather than erasing them to a single type.


## Adding a feature

- Write a new component in full in its own files. Do not refactor existing, working code to share helpers with it; duplicate similar setup code if needed, and touch existing code only where the feature genuinely requires it.
- Every new feature ships with tests in the package's test target, and the existing suites of every package touched are run.
- Every new feature also ships with an example that showcases it. A small addition goes into an existing example where it fits; a larger one gets its own app.
- An example is a real, usable app of the kind the feature implies, with the controls a user expects: a web view's example is a browser with an editable address field, back, forward, reload, and tabs, not a row of preset buttons.
- A library's example apps live in `Examples/<Name>/`, each its own package that depends on the library through `.package(path: "../..")`. Never add an example as an executable target in the library's own `Sources/`.
- Capability types follow `platforms.md`, with one file per platform.
