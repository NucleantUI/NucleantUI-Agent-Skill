---
name: nucleantui-pro
description: Comprehensively reviews NucleantUI code for best practices on its API, cross-platform rules, maintainability, and performance. Use when reading, writing, or reviewing NucleantUI apps, examples, or libraries built on NucleantUI.
license: MIT
metadata:
  author: NucleantUI
  based-on: SwiftUI Pro by Paul Hudson (https://github.com/twostraws/SwiftUI-Agent-Skill)
  version: "1.0.0"
---

Review Swift and NucleantUI code for correctness, current API usage, and adherence to project conventions. Report only genuine problems – do not nitpick or invent issues.

Include deprecated API in review findings. During feature work or bug fixes, complete the requested change and suggest unrelated modernization separately; do not apply it without being asked.

Review process:

1. Check that every view struct uses `@View` and is declared correctly using `references/views.md`.
1. Check for API NucleantUI does not have, deprecated API, and SwiftUI habits that do not carry over using `references/api.md`.
1. Validate that data flow is configured correctly using `references/data.md`.
1. Ensure navigation, toolbars, and presentations use what NucleantUI provides using `references/navigation.md`.
1. Check that the code builds and behaves the same on every platform using `references/platforms.md`.
1. Check drawing, shaders, `ThorCanvas`, and image rendering using `references/rendering.md`.
1. Ensure the design is consistent, stable, and usable on mouse and touch using `references/design.md`.
1. Review layout behavior as windows resize and safe areas change using `references/resizability.md`.
1. Check what accessibility the code can offer today using `references/accessibility.md`.
1. Ensure the code is able to run efficiently using `references/performance.md`.
1. Quick validation of Swift code using `references/swift.md`.
1. Final code hygiene check using `references/hygiene.md`.

If doing a partial review, load only the relevant reference files.

Keep `references/performance-plus.md` out of routine reviews. Load it for a requested deep performance review; its more involved changes can also help investigate an established performance problem.

Load `references/framework.md` only when the code is NucleantUI's own source, or a library that extends NucleantUI.


## Core Instructions

- NucleantUI is a SwiftUI-shaped UI framework: views drawn by Skia on Vulkan, hosted by NucleantApplication, with no AppKit/UIKit view hierarchy. If you know SwiftUI you know most of its surface, but it is not SwiftUI: `import SwiftUI` never appears.
- Every view struct gets `@View`. `struct Foo: View` is always a finding, never a judgment call.
- NucleantUI implements a subset of SwiftUI. Before recommending any SwiftUI API, confirm NucleantUI declares it – search the NucleantUI sources the project builds against (for a package, `.build/checkouts/NucleantUI/Sources/NucleantUI`). Never recommend or invent API it does not have, and do not flag the absence of an API as a problem in the user's code.
- When the user specifies an API or design, implement exactly that. If it hits a technical blocker, say so and propose fixes within their design rather than substituting your own.
- Code must build and behave the same on macOS, iOS, Linux, Android, Windows, and WebAssembly. App code never imports Apple-only frameworks and never branches on the platform.
- Target Swift 6.3 (`// swift-tools-version: 6.3`), using modern Swift concurrency with strict checking.
- Check the package's `platforms:` before using API that Apple platforms gate by OS version; keep the deployment targets unchanged.
- Do not introduce third-party frameworks without asking first.
- Break different types up into different Swift files rather than placing multiple structs, classes, or enums into a single file.
- Use a consistent project structure, with folder layout determined by app features.


## Output Format

Organize findings by file. For each issue:

1. State the file and relevant line(s).
2. Name the rule being violated (e.g., "Put `@View` on every view struct").
3. Show a brief before/after code fix.

Skip files with no issues. End with a prioritized summary of the most impactful changes to make first.

Example output:

### ScoreBadge.swift

**Line 3: Put `@View` on every view struct; `struct ScoreBadge: View` is not allowed.**

```swift
// Before
struct ScoreBadge: View {

// After
@View
struct ScoreBadge {
```

**Line 12: `animation(_:)` without a value is deprecated – give it the value to watch.**

```swift
// Before
Text("\(score)").scaleEffect(isPulsing ? 1.2 : 1).animation(.bouncy)

// After
Text("\(score)").scaleEffect(isPulsing ? 1.2 : 1).animation(.bouncy, value: isPulsing)
```

### Panel.swift

**Line 9: A hand-written initializer must take and store `_viewID`, or a panel made outside a builder has no call-site identity.**

```swift
// Before
init(_ title: String, @ViewBuilder content: () -> Content) {
    self.title = title
    self.content = content()
}

// After
init(_ title: String, _viewID: ViewID = #viewID, @ViewBuilder content: () -> Content) {
    self.title = title
    self.content = content()
    self._viewID = _viewID
}
```

The `@View` macro warns about this initializer and offers the same fix-it.

### Summary

1. **View declaration (high):** `ScoreBadge` is missing `@View`.
2. **View identity (high):** `Panel.init` drops the call site, so state can attach to the wrong panel.
3. **Deprecated API (medium):** The unvalued `animation()` on line 12 animates every change to the view.

End of example.


## References

- `references/accessibility.md` - what NucleantUI offers for accessibility today, and the habits that still apply.
- `references/api.md` - SwiftUI API that is missing, renamed, or different in NucleantUI, plus deprecated API.
- `references/data.md` - data flow, `@Observable` models, `@State`, bindings, and identity.
- `references/design.md` - consistent, stable designs that work with mouse and touch, in light and dark.
- `references/framework.md` - working on NucleantUI itself or a library that extends it.
- `references/hygiene.md` - making code compile cleanly and be maintainable in the long term.
- `references/navigation.md` - `NavigationStack`, toolbars, popovers, context menus, and missing presentations.
- `references/performance.md` - optimizing NucleantUI code, and measuring it.
- `references/performance-plus.md` - deeper restructuring to narrow what views depend on, reserved for deep performance reviews or diagnosed performance issues.
- `references/platforms.md` - cross-platform capability types, Foundation off Darwin, and per-platform files.
- `references/rendering.md` - Skia on Vulkan, `ThorCanvas`, PyShader shaders, and rendering views to images.
- `references/resizability.md` - changing window sizes, adaptive layouts, and safe areas.
- `references/swift.md` - tips on writing modern Swift code, including using Swift Concurrency effectively.
- `references/views.md` - `@View`, view structure, composition, and animation.
