# Platforms

NucleantUI, and every app, example, and demo built on it, targets macOS, iOS, Linux, Android, Windows, and WebAssembly, plus whatever gets added later. A file that only compiles on Apple platforms is broken, whether it is framework code or an example.


## App code never sees the platform

- Views, examples, and demos never import an Apple-only framework – `AVFoundation`, `AudioToolbox`, `CoreAudio`, `CoreMIDI`, `CoreText`, `CoreGraphics`, `ImageIO`, `AppKit`, `UIKit`, `Metal`, `Combine`, `SwiftData`, `WebKit`, and so on – and never contain `#if os(...)` or `#if canImport(...)` to reach one. Flag every instance.
- Anything that needs the OS – loading a font, decoding an image, loading or playing audio, MIDI, file pickers – is a concrete type with one API on every platform. Callers use that type and never see the platform. The framework already provides some: `RasterImage(contentsOf:)` decodes images on every platform, and `FontRegistry.register(path:as:)` makes a font file a `Font.custom` family.
- If an app needs a capability NucleantUI does not have yet, build the cross-platform type first, then use it. Do not wire a platform framework straight into a view or a model.


## Building a capability type

- One shared file declares the type, or a protocol, and the API every platform provides. Each platform gets its own file – `Foo+MacOS.swift`, `Foo+iOS.swift`, `Foo+Linux.swift`, `Foo+Android.swift`, `Foo+Windows.swift`, `Foo+Wasm.swift` – wrapped whole in its `#if os(...)`, importing that platform's frameworks and adding the matching extension.
- When the implementation needs its own type per platform, each platform file defines it under the same name, so only one is in scope per build and the shared code refers to it without any `#if`. Tie them together with protocols and generics, not existentials.
- `#if` goes at the top of those platform files, not through shared code, and never in view bodies or examples. Guarding alone, without a type that hides the platform, is not enough.
- Every platform gets its file. Decide what each one does before writing the Apple ones: a real native backend, a portable implementation shared by all, or – only if neither is practical yet – a file that compiles and fails visibly with a thrown "unsupported on this platform" error. A silent no-op or a missing file is a finding.
- Design the public API from what all platforms can do. Never put platform types such as `AVAudioPCMBuffer`, `CTFont`, `CGImage`, or `CGFloat` in its signatures.
- A capability that represents a device, resource, or changing state – a camera, a flashlight's on/off state, a permission, a photo library – is an `@Observable` class that keeps the live platform object and changes through its own methods. Do not copy its values out into structs.
- An enum mirroring a platform's integer constants uses them as raw values – `enum BatteryState: Int32 { case unknown = 1; case charging = 2 }`, created with `BatteryState(rawValue: x)` – not an `init(code:)` switch or a computed mapping.


## Foundation off Darwin

Foundation is available everywhere, but not every API behaves the same outside Apple platforms. Check before relying on something Darwin-specific.

- Android's Foundation brings in only the C library's `Double` math functions; the `Float` overloads of `sqrt`, `pow`, `tanh`, `cos`, and the rest are missing there. Do math in `Double` and convert to `Float` at the end.
- On WebAssembly the page owns the event loop, so a Foundation `Timer` has no run loop to fire it. Tick with a task that loops over `try await Task.sleep(for: .seconds(1))`, or use `TimelineView`, which work on every platform.
- A build failing on one platform shows up only in that platform's build. Before calling cross-platform code finished, build it for Android and WebAssembly too.
