# Performance

## Measuring

- Measure with the framework's tracing, set in the environment when running the app: `NUCLEANT_SWIFTUI_TRACE_PERF=1` prints each rebuild with the views built and reused and the render nodes repainted; `=2` adds every view built or reused with the reason it was not reused. A control being dragged should repaint one or two nodes and not the window. `NUCLEANT_SWIFTUI_TRACE_LAYOUT=1` prints the placed display list and each node's frame and image size.
- Measure real interactions from outside. Never add benchmark hooks, forced repaints, or timing code to the app to measure it.


## Keeping views cheap

- Keep conditional modifier values inside the modifier, for example `.opacity(isUnavailable ? 0.4 : 1)`, rather than branching the view with `if`/`else`. Branching produces `_ConditionalContent`, which changes structural identity and rebuilds the branch, resetting its state, each time the condition flips. Never introduce a `View.if()` helper that conditionally transforms its receiver. Report existing helpers with this explanation, but leave their removal to a requested change because they may have many callers.
- `AnyView` is a last resort. Use a generic parameter (`struct Card<Content: View>`) for one view of a caller-chosen type, and a parameter pack (`struct Row<each Content: View>`) for a fixed-arity list of different views. Reserve `AnyView` for a genuinely unbounded, heterogeneous boundary, such as a closure stored as a property that different call sites fill with different view types.
- Never use `any View` or `Any` to hold views; they have no accepted use, last resort or otherwise.
- For view extraction, follow the guidance and exceptions in `views.md`.
- Always ensure view initializers are kept as small and simple as possible, avoiding any non-trivial work. Flag any work that can be moved into a `task()` modifier to be run when the view is shown.
- Similarly, assume each view's `body` property is called frequently – if logic such as sorting or filtering can be moved out of there easily, it should be.
- Avoid creating properties to store formatters such as `DateFormatter` unless they are required. Format with `FormatStyle` where the text is made: `Text(date.formatted(.dateTime.day().month().year()))`.
- Avoid expensive inline transforms in `List`/`ForEach` initializers (e.g. `items.filter { ... }`) when they are repeated often.
- Prefer deriving transformed data from the source of truth using `let`, or caching it in `@State`. However, do not cache derived collections in `@State` unless you also own explicit invalidation logic to avoid stale UI.
- For large data sets in `ScrollView`, use `LazyVStack`/`LazyHStack`; flag eager stacks with many children.
- Prefer using `task()` over `onAppear()` when doing async work, because it is cancelled automatically when the view leaves the tree.


## View inputs

A parent rebuilding keeps a child whose inputs are equivalent to last time. Anything that makes inputs compare as changed rebuilds that child, and its subtree, every time.

- Pass a `Binding` projected from `@State` or `@Binding` rather than a `Binding(get:set:)`, which never compares equal. See `data.md`.
- A stored closure is compared by the code it runs and the values it captured. Capturing something with no way to compare it – a captured `var`, or a non-`Equatable` struct that holds a reference – makes the view count as changed on every rebuild. Prefer a `Binding` or a value, or capture only comparable values.
- Avoid storing escaping `@ViewBuilder` closures on views; store the built view instead, as the `Panel` example in `views.md` does.

```swift
// Anti-pattern: stores the builder closure on the view.
@View
struct Card<Content: View> {
    let content: () -> Content

    var body: some View {
        VStack(alignment: .leading) {
            content()
        }
        .padding()
        .background(.secondaryBackground)
        .cornerRadius(8)
    }
}
```

Store `let content: Content` and call the builder once in a hand-written `init` that also takes `_viewID`.


## Collections and row updates

- Pass each row its element, such as `BookmarkRow(bookmark: bookmark)`. Passing a store and a position or identifier makes the row search the collection, creating a dependency on the entire collection so one element's changes can update all rows.
- If rows own separate observable models, create and retain those models once. Constructing `BookmarkRow(model: BookmarkModel(bookmark: bookmark))` inside `body` or a `ForEach` callback gives every row a different object on each update.


## Frequent changes

- Store rapidly changing measurements – scroll positions, geometry, drag coordinates, animation progress, timer ticks – in an `@Observable` object, and expose separately stored, less frequently changing results, such as `hasReachedEnd`, for views to read. Converting the raw measurement to a Boolean inside `body` still subscribes that view to every raw update.
- Use `scrollTransition()` or `visualEffect()` for scroll-driven appearance changes such as fading, scaling, and rotation, rather than reading the scroll position into state. Keep state when scroll position also drives application logic.
- Remove unread `@Environment` properties; they are unnecessary code.
