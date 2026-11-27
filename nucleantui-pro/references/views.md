# NucleantUI Views

## Declaring views with `@View`

- Every view struct gets `@View`, with zero exceptions: top-level screens, tiny private helpers, everything. `struct Foo: View` is forbidden; seeing it means fixing it, not evaluating it.

```swift
@View
struct TrackRow {
    let name: String
    @Binding var level: Double
    @State private var isExpanded = false

    var body: some View { ... }
}
```

- `@View` adds the `View` conformance plus what the framework uses to avoid rebuilding a view: an identity for the place the view was constructed (`_viewID`), a static list of its `@State`/`@Binding`/`@Environment` properties, and a comparison over its inputs. When a parent rebuilds and produces an equivalent child at the same position, the child's body is not evaluated.
- `@View` applies to structs only.
- When the struct declares no initializer, `@View` generates one from the stored properties, ending with `_viewID: ViewID = #viewID`. A `let` with a value is fixed, a `var` with a value becomes a defaulted parameter, a `@Binding` takes the binding, a `@Bindable` takes the object, and a `@State` with an initial value is not a parameter.
- A hand-written initializer must take `_viewID: ViewID = #viewID` as its last parameter before any trailing closure, and store it with `self._viewID = _viewID`. Otherwise a view constructed outside a `@ViewBuilder` body has no call-site identity. The macro warns about such initializers and offers a fix-it.
- Never write `return` in a `body` or any builder closure. A `let` before the view expression is a declaration, not part of the result, and needs no `return`; do not restructure code into computed properties or extra views to avoid the `let`.

```swift
var body: some View {
    let progress = Double(done) / Double(max(total, 1))
    ProgressRing(progress: progress)
}
```

- A container view stores its built content, not the builder closure, and takes the closure in its own initializer:

```swift
@View
struct Panel<Content: View> {
    let title: String
    let content: Content

    init(_ title: String, _viewID: ViewID = #viewID, @ViewBuilder content: () -> Content) {
        self.title = title
        self.content = content()
        self._viewID = _viewID
    }

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(title).font(.headline)
            content
        }
    }
}
```


## Structuring views

- Strongly prefer to avoid breaking up view bodies using computed properties or methods that return `some View`. Extract them into separate `@View` structs instead, placing each into its own file. An extracted view is compared on its inputs and skipped when they are unchanged; a computed property runs again every time its parent's body does, and its reads become the parent's.
- Flag `body` properties that are excessively long; they should be broken into extracted subviews.
- If the user has created a handful of small, private helper `some View` properties for structural readability, and they both belong to the same concern as `body` and would fit in `body` at an acceptable length if inlined, these can be left alone. Otherwise, they should be extracted to new `@View` structs.
- Button actions should be extracted from view bodies into separate methods, to avoid mixing layout and logic.
- Similarly, general business logic should not live inline in `task()`, `onAppear()` or elsewhere in `body`.
- Prefer to place view logic into `@Observable` models, so it can be tested. For more help with testing, suggest the [Swift Testing Pro agent skill](https://github.com/twostraws/swift-testing-agent-skill).
- Each type (struct, class, enum) should be in its own Swift file. Flag files containing multiple type definitions.
- Unless a full-screen editing experience is required, prefer `TextField` with `axis: .vertical` to `TextEditor`, because it allows placeholder text. If a specific minimum height is required, use something like `lineLimit(5...)`.
- If a button action can be provided directly as an `action` parameter, do so: `Button("Add", action: addItem)` is preferred over `Button("Add") { addItem() }`.
- Remove a `Group` that merely wraps one view; attach its modifiers to that view directly. Groups containing siblings, a `ForEach`, or conditional branches serve a purpose and should stay.
- `.scaleEffect()` is for animation. Never use it to make content fit its frame; size the content with frames and layout, and let texture sources render at the texture's own pixel size.


## Missing views

- When a screen needs a view NucleantUI does not have yet, such as `TabView`, build a real one in the app's own package, following SwiftUI's API for it. Never fake it with something else, such as a segmented `Picker` standing in for tabs.
- When a window runs out of room for controls, split them into tabs or pages rather than wedging panels into leftover corners.


## Animating views

- Never use `animation(_ animation: Animation?)`, which NucleantUI deprecates; always provide a value to watch, such as `.animation(.bouncy, value: score)`, or use `withAnimation`.
- Chaining animations must be done using a `completion` closure passed to `withAnimation()`, rather than trying to execute multiple `withAnimation()` calls using delays.
- There is no `@Animatable` macro. Conform to `Animatable` and expose `animatableData`; combine two values with `AnimatablePair`.

For example:

```swift
Button("Animate Me") {
    withAnimation {
        scale = 2
    } completion: {
        withAnimation {
            scale = 1
        }
    }
}
```
