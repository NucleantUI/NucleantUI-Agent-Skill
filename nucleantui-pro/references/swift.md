# Swift

- Prefer Swift-native string methods over Foundation equivalents: use `replacing("a", with: "b")` not `replacingOccurrences(of: "a", with: "b")`.
- Prefer modern Foundation API, such as `appending(path:)` to append strings to a URL – but check that it behaves the same off Darwin; see `platforms.md`.
- Never use C-style number formatting like `String(format: "%.2f", value)`. Use `FormatStyle`: `value.formatted(.number.precision(.fractionLength(2)))`.
- Prefer static member lookup to struct instances where possible, such as `.toggleStyle(.switch)` rather than `.toggleStyle(SwitchToggleStyle())`, and `.listStyle(.plain)` rather than `.listStyle(PlainListStyle())`.
- Avoid force unwraps (`!`) and force `try` unless the failure is truly unrecoverable, and even then prefer using `fatalError()` with a clear description. If possible, use `if let`, `guard let`, nil-coalescing, or `try?`/`do-catch`.
- Filtering text based on user input must be done using `localizedStandardContains()` as opposed to `contains()` or `localizedCaseInsensitiveContains()`.
- Use `Double` for geometry and math; NucleantUI's geometry is `Double` throughout, and `CGFloat` is not part of its API. Convert to `Float` only where an API requires it, such as shader arguments.
- If you want to count array objects that match a predicate, always use `count(where:)` rather than `filter()` followed by `count`.
- Prefer `Date.now` over `Date()` for clarity.
- When dealing with the names of people, strongly prefer to use `PersonNameComponents` with modern formatting over simple string interpolation such as `"\(firstName) \(lastName)"`.
- If a given type of data is repeatedly sorted using an identical closure, e.g. `books.sorted { $0.author < $1.author }`, prefer to make the type in question conform to `Comparable` so the sort order is centralized.
- Prefer modern date formatting over handwritten patterns. If user-visible dates require `DateFormatter`, configure it with `setLocalizedDateFormatFromTemplate()` rather than assigning `dateFormat`, so the locale determines component ordering. Fixed formats used for API exchange are outside this rule.
- Format lists for display with `formatted()` rather than inserting comma separators yourself. For example, `destinations.formatted()` provides the locale's punctuation and conjunction. Keep `joined(separator:)` for storage or interchange where a fixed delimiter is intentional.
- When trying to convert a string to a date, prefer the modern `Date` initializer API such as `try Date(myString, strategy: .iso8601)`.
- Flag instances where errors triggered by a user action are swallowed silently, e.g. using `print(error.localizedDescription)` rather than showing the problem in the interface.
- Prefer `if let value {` shorthand over `if let value = value {`.
- Omit `return` for single expression functions. `if` and `switch` can be used as expressions when returning values and assigning to variables.

For example, this kind of code:

```swift
var tileColor: Color {
    if isCorrect {
        return .green
    } else {
        return .red
    }
}
```

Should be written like this:

```swift
var tileColor: Color {
    if isCorrect {
        .green
    } else {
        .red
    }
}
```


## Types

- Never declare a type with the same name as its module, such as `enum MyLibrary` in module `MyLibrary`. Swift uses the module name as a namespace, so the type shadows it and qualified lookups such as `MyLibrary.Foo` break.
- Do not write an `enum` full of static functions as a namespace. A group of related functions with setup or state is a `final class` you instantiate, with instance properties and methods. Real enums with cases, error enums, and tables of constants are fine.
- Do not add a type – a token, handle, or wrapper class – or a protocol requirement to carry something small. Use what the API already passes: a `Bool` return, a closure the caller already gives, or a parameter at the call site.
- Reach for generics and parameter packs before existentials. Plain `any Protocol` and `Any` for type erasure are a last resort; for reference types, prefer manual pointer-based erasure (`Unmanaged`, `UnsafeMutableRawPointer`) to boxing a class instance in `Any`. Views are erased only with `AnyView`; see `performance.md`.


## Swift Concurrency

- If an API offers both modern `async`/`await` equivalents and older closure-based variants, prefer the `async`/`await` versions.
- Never use Grand Central Dispatch (`DispatchQueue.main.async()`, `DispatchQueue.global()`, etc.). Always use modern Swift concurrency (`async`/`await`, actors, `Task`).
- Never use `Task.sleep(nanoseconds:)`; use `Task.sleep(for:)` instead.
- Flag any mutable shared state that isn't protected by an actor or `@MainActor`, unless the project is configured to use MainActor default actor isolation.
- Assume strict concurrency rules are being applied; flag `@Sendable` violations and data races.
- When evaluating `MainActor.run()`, check whether the project has its default actor isolation set to Main Actor first, because `MainActor.run()` might not be needed.
- `Task.detached()` is often a bad idea. Wherever it is used, check that it is the best approach.

For more help with Swift concurrency, suggest the [Swift Concurrency Pro agent skill](https://github.com/twostraws/swift-concurrency-agent-skill).
