# Data flow, shared state, and property wrappers

It is important that view body code and logic code be kept separate in order to make code easier to read, write, and maintain. That usually means placing code into methods rather than inline in the `body` property, but often also means carving functionality out into separate `@Observable` classes.

These rules help ensure code is efficient and works well in the long term.


## Models are `@Observable` classes

- App data that changes over time – a session, a document, a list being edited, anything the user works on – is a `@MainActor @Observable final class`, changed through its own methods. It is not a struct held in `@State` and replaced or mutated wholesale through a `@Binding`. This applies to examples and demos too: they are the code people copy.
- The view that owns the model holds it in `@State`. Views below take it as a plain `let`, or as `@Bindable` when they need a binding into one of its properties. To reach a model deep in the tree, use `.environment(model)` and `@Environment(Model.self)`.
- Structs are for the small values inside a model – an entry, a record, an input – and for view inputs.
- A view's own transient UI state, such as whether a panel is open or how far something is dragged, stays in that view's `@State`; it is not model data.
- `@Observable` is the standard library's: add `import Observation`.
- `@Observable` classes must be marked `@MainActor` unless the project has Main Actor default actor isolation.
- Where possible, give custom value types stored by an `@Observable` class `Equatable` conformance. Observation can then skip notifications for assignments equal to the current value.

```swift
import NucleantUI
import Observation

@MainActor @Observable
final class Playlist {
    private(set) var tracks: [Track] = []

    func rate(_ id: Track.ID, _ rating: Double) {
        guard let index = tracks.firstIndex(where: { $0.id == id }) else { return }
        tracks[index].rating = rating
    }
}

@View
struct PlaylistScreen {
    @State private var playlist = Playlist()

    var body: some View {
        TrackList(playlist: playlist)
    }
}
```

Every property a `body` reads is tracked, and a write rebuilds only the views that read it. For an `@Observable` model, Observation records the stored properties a view accesses: reading a computed property records every stored property its getter uses, and a stored struct is one dependency, so reading `library.book.title` also subscribes to `library.book.pageCount`. `@State` is tracked at key-path granularity: a write through `$tracks[3].level` rebuilds the view that owns the state and the views that read that part, not every row. Structural changes to narrow model dependencies belong in a deep review; see `performance-plus.md`.


## Local state

- Declare view-owned `@State` as `private`. It is owned, not received: `@View` never makes a `@State` with an initial value an initializer parameter. A `@State` with no initial value does become a parameter; to let callers choose a starting value for one that has a default, write that initializer yourself.
- `@State` is a macro whose initial expression runs once, when the state is created, not each time the struct is constructed. `@State private var model = Model()` runs `Model()` once for that place in the tree, even when the parent rebuilds every frame.
- Initialize each `@State` property in one place. Assigning a `@State` that has an initial value in `init()` replaces that value only the first time, when the state is created; later constructions ignore it.
- If a view stores a class instance that contains expensive-to-recompute data, it can be stored using `@State` even though it is not observable. This uses `@State` as a cache – storing something persistently, without change tracking.
- In apps with more than one window (`WindowGroup`, `openWindow`), keep window-specific UI state such as the selection or navigation path in that window's root view `@State`. Singletons and shared models couple the windows so they cannot navigate independently.


## Bindings

- Prefer bindings projected from `@State` and `@Binding`, including key-path bindings such as `$tracks[index].level`. They carry their source, so a child handed one is still an unchanged input when the parent rebuilds. A `Binding(get:set:)` has no source and never compares equal, so a child that receives one is rebuilt every time its parent is.
- Use `@Bindable` key-path bindings, such as `$model.gain`, to bind a control directly into a model property.
- Preserve behavior when refactoring bindings. A setter handles writes through that binding, including equal-value assignments; `onChange()` responds to value changes, including programmatic ones. Leave a custom binding intact when it routes a write through a model method and no projected equivalent exists.
- If the user needs to enter a number into a `TextField`, bind it to a numeric value such as `Int` or `Double` with the `format` initializer: `TextField("Score", value: $score, format: .number)`.


## Working with data

- Prefer to make structs conform to `Identifiable` rather than using `id: \.someProperty`.
- In mutable collections displayed by `ForEach`, `List`, or `Table`, identity must come from the element. Indices and offsets describe positions, so inserts, deletes, moves, and filtering can reset row state and disrupt animations. Avoid `ForEach(books.indices, id: \.self)` and `enumerated()` identified by `\.offset`; use `ForEach(Array(books.enumerated()), id: \.element.id)` when the position is needed. Fixed ranges and constant collections are not a problem.
- Allocate an element's identity when storing the element, not while evaluating `body`. Mapping labels to fresh values inside `ForEach` recreates every row if each initializer generates a `UUID`. A stored `let id = UUID()` is valid when its containing value survives body evaluations in `@State` or a model.
- Use a compact, immutable key to identify objects. Avoid `id: \.self` for a full struct, which hashes every field for every row on updates; reserve it for small unique values such as integers, UUIDs, URLs, or short strings. Do not derive identity from editable fields, such as `var id: String { caption }`: changing the caption then replaces the row and loses focus or selection.
