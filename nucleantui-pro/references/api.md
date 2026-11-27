# Using NucleantUI API

NucleantUI follows SwiftUI's names, signatures, and behavior, but implements a subset of it, with a few deliberate differences. Code written from SwiftUI memory often does not compile here, or compiles and means something else. When a symbol's status is unclear, read its declaration in the NucleantUI sources the project builds against rather than assuming SwiftUI's.

Also check the animation rules in `views.md` and the navigation rules in `navigation.md`.


## Deliberate differences

- Views are declared with `@View`, not by conforming to `View`. See `views.md`.
- `MenuBarExtra` is `SystemTray`, with `.systemTrayStyle(.menu)` / `.window`; its initializers are otherwise SwiftUI's, such as `SystemTray("Timer", systemImage: "timer") { ... }`.
- `ImageRenderer` is `renderImage(size:scale:)`, and `.image(size:scale:)` on a view; both return a `RasterImage`.
- NucleantUI additions with no SwiftUI equivalent are correct API, not mistakes: `.relativeSize(width:height:)` (a fraction of what the parent offers), `.framed(_:alignment:)`, `PathShape`, `ArrowBack`, `Color.dynamic(light:dark:)`, `FontRegistry`, `SvgView`/`SvgImage`, `ThorCanvas`, `RenderTexture`, `TextureView`, and `ShaderProtocol`.


## SwiftUI habits that do not carry over

- `import SwiftUI` never appears; import `NucleantUI`, and `Observation` for `@Observable`.
- Views have no asset catalogs, SF Symbols, `Image("name")`, or `Image(systemName:)`. A bitmap is a `RasterImage` (for example `RasterImage(contentsOf: url)`) shown with `Image(_:)`; an icon is an SVG shown with `SvgView`, using `.renderingMode(.template)` to paint it in the foreground style.
- `Text` takes a `String`, shown verbatim. There is no `LocalizedStringKey`, `Text` concatenation, Markdown, or `Text(_:format:)`. Format values into the string with `FormatStyle`, such as `Text(price.formatted(.currency(code: "EUR")))`.
- `Button` has `Button(_ title:action:)` and `Button(action:label:)` only; there is no `systemImage:` or `role:` initializer, and no `Label` view.
- There is no `ObservableObject`, `@Published`, `@StateObject`, `@ObservedObject`, or `@EnvironmentObject`, and Combine does not exist off Apple platforms. Use `@Observable`; see `data.md`.
- There is no `@Entry` macro. Define a custom environment value with an `EnvironmentKey` and an `EnvironmentValues` extension.
- `.onChange()` has the two-parameter form `{ oldValue, newValue in }` and the zero-parameter form, both with `initial:`. Use one of those.
- A shape cannot be filled and stroked by chaining: `fill()` returns a view. Overlay a stroked copy instead: `Circle().fill(.orange).overlay { Circle().stroke(.white, lineWidth: 2) }`.
- `clipShape()` honors only a shape's bounding box and corner rounding (`RoundedRectangle`, `Capsule`, `Circle`, `Ellipse`); it is not a path clip. There is no `.rect(cornerRadius:)` static shape; write `RoundedRectangle(cornerRadius: 12)`, or use `.cornerRadius(12)`, which is current API here.
- Geometry is `Double` throughout (`Size`, `Rect`, frames, offsets). Never use `CGFloat`, `CGPoint`, `CGSize`, or `CGRect`.


## Current API

- Prefer `foregroundStyle()` to `foregroundColor()` in new code: it takes any `ShapeStyle`, including `.secondary`, `.tint`, `Color.dynamic(light:dark:)` pairs, and gradients. `foregroundColor()` is not deprecated in NucleantUI, so existing uses are not findings.
- When a style ternary fails because its branches have different types, erase each style with `AnyShapeStyle`: `.foregroundStyle(isSelected ? AnyShapeStyle(.white) : AnyShapeStyle(.secondary))`.
- Never use `.navigationBarLeading` or `.navigationBarTrailing`. Prefer placements that express purpose – `.cancellationAction`, `.confirmationAction`, `.primaryAction`, `.destructiveAction`, `.secondaryAction` – and use `.topBarLeading` or `.topBarTrailing` only when no semantic role fits.
- Use `toolbarVisibility(.hidden, for: .navigationBar)` to hide a bar, rather than the visibility overload of `toolbar(_:for:)`.
- Use `.environment(model)` and `@Environment(Model.self)` to share an `@Observable` model through the tree.
- For sizing and positioning, prefer `.relativeSize()`, `visualEffect()`, `onGeometryChange()`, or the `Layout` protocol to `GeometryReader` when they can do the job.
- When hiding the status bar on iOS and Android, use `statusBarHidden()`; other platforms have no status bar, and it does nothing there.
- Use `scrollPosition()` and `ScrollViewReader` for programmatic scrolling, and `scrollTransition()` for appearance changes driven by scroll position.
