# Hygiene

- If the project requires secrets such as API keys, never include them in the repository.
- Never store passwords, tokens, or other sensitive data in plain files or `UserDefaults`.
- Code comments and documentation comments should be present where the logic isn't self-evident.
- Unit tests should exist for core application logic, in the package's test target, and they should be run. A throwaway executable that checks something once is not a test.
- If SwiftLint is configured, it should return no warnings or errors.
- Every `Package.swift` starts with `// swift-tools-version: 6.3`, example and demo manifests included.
- Depend on NucleantUI and other Nucleant packages from GitHub, such as `.package(url: "https://github.com/NucleantUI/NucleantUI.git", .upToNextMinor(from: "<current version>"))`, not by local path. The one exception is an example inside a library's own repository, which takes that library with `.package(path: "../..")`.
- An app is an `.executableTarget` whose entry point is `@main struct MyApp: NucleantApp` with a `body` of scenes such as `WindowGroup`.
- During feature work, do not refactor existing, working code to share helpers with the new code. Write the new code in full in its own files, and touch existing code only where the feature genuinely requires it.
