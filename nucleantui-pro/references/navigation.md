# Navigation and presentation

- Use `NavigationStack`. `NavigationView` does not exist, and neither does `NavigationSplitView` yet.
- Strongly prefer `NavigationStack(path:)` with `NavigationLink(value:)` and `navigationDestination(for:)` when navigation is driven by data. `NavigationLink(_:destination:)` remains correct for a fixed screen that needs no data.
- Register `navigationDestination(for:)` once per data type within each navigation stack.
- Title the root with `NavigationStack("Library") { ... }` or `.navigationTitle()`, and pushed screens with `.navigationTitle()`.
- Screens beneath the top one stay alive, keeping their state and scroll positions. Do not add code that saves and restores them around a push.
- For a back button of your own, apply `.navigationBarBackButtonHidden()`, put the replacement in the `.navigation` placement, and pop through `@Environment(\.navigationRouter)`. `ArrowBack` is the stack's own chevron as a shape.


## Toolbars

- Toolbar items appear in the stack's own bar on every platform, macOS included, never in the window's title bar. Use the semantic placement rules in `api.md`.
- `.secondaryAction` items go behind a "…" menu that the bar provides; do not build a custom ellipsis `Menu` for them.
- Place related controls, such as previous and next actions, in a `ToolbarItemGroup` instead of separate items divided by spacers.
- `.bottomBar` and `.status` items go in a bar along the bottom. Prefer them, and the stack's bar, to hand-built bars.
- Hide a bar with `toolbarVisibility(.hidden, for: .navigationBar)`. A root screen with no title, items, or back button already has no bar.


## Popovers, menus, and missing presentations

- `.popover(isPresented:)` presents a panel of ordinary views pointing at its source, kept inside the window. A press outside it, or Escape, sets the binding to `false`; the content can also set it `false` itself.
- `.contextMenu()` opens on a right click or a press held still on touch hosts. Its rows are `Button`s, a nested `Menu` is a submenu, and `Divider` is a rule. A standalone `Menu` is a button that opens its items beneath itself.
- There is no `sheet()`, `fullScreenCover()`, `alert()`, `confirmationDialog()`, or `inspector()` yet. Use a popover when it fits. Otherwise build the missing presentation as a real, reusable modifier with SwiftUI's API shape, such as `.sheet(isPresented:content:)`, in the app's own package – not as an ad-hoc `ZStack` and offset inside one screen.
- For an app's menu bar or system tray icon, use `SystemTray` in the app's scenes.
