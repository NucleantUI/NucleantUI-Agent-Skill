# NucleantUI Agent Skill for AI Coding Assistants

An agent skill that helps AI coding assistants write correct, simple, cross-platform [NucleantUI](https://github.com/NucleantUI/NucleantUI) code. It covers `@View`, data flow, navigation, layout, rendering and shaders, performance, and the rules that keep one codebase building on macOS, iOS, Linux, Android, Windows, and WebAssembly. It targets the mistakes LLMs actually make when they write NucleantUI from SwiftUI memory.

It uses the [Agent Skills](https://agentskills.io/home) format, so it works with Claude Code, Codex, Gemini, Cursor, and more.


## Based on SwiftUI Pro by Paul Hudson

This skill is a NucleantUI port of **[SwiftUI Pro](https://github.com/twostraws/SwiftUI-Agent-Skill)** by [Paul Hudson](https://www.hackingwithswift.com), who writes [free Swift tutorials at Hacking with Swift](https://www.hackingwithswift.com). Its structure, review process, output format, and many of its rules are his work, adapted here to NucleantUI's API and platforms. Thank you, Paul.

If you write SwiftUI rather than NucleantUI, use his original. You can read why he created it in [SwiftUI Agent Skill - Write better code with Claude, Codex, and other AI tools](https://www.hackingwithswift.com/articles/282/swiftui-agent-skill-claude-codex-ai). His [Swift Concurrency Pro](https://github.com/twostraws/Swift-Concurrency-Agent-Skill) and [Swift Testing Pro](https://github.com/twostraws/Swift-Testing-Agent-Skill) skills apply to NucleantUI projects as they are.

What changed in the port:

- Every view is declared with `@View`, including the rules for hand-written initializers and `_viewID`.
- Rules for SwiftUI API that NucleantUI does not have were removed or replaced, and NucleantUI's own differences (`SystemTray`, `renderImage`, `RasterImage`, SVG icons, `Double` geometry) were added.
- New references cover cross-platform capability types, Foundation off Darwin, Skia on Vulkan, `ThorCanvas`, PyShader shaders, and work on NucleantUI itself.
- Performance guidance uses NucleantUI's own tracing and how it compares view inputs.


## Installing NucleantUI Pro

You can install this skill into Claude Code, Codex, Gemini, Cursor, and more by using npx:

```bash
npx skills add https://github.com/NucleantUI/NucleantUI-Agent-Skill --skill nucleantui-pro
```

When using `npx`, you can select exactly which agents you want to use during the installation, and whether the skill is installed for one project or for all of them.

Claude Code users can add the marketplace and install the plugin directly through Claude:

```
/plugin marketplace add NucleantUI/NucleantUI-Agent-Skill
/plugin install nucleantui-pro@nucleantui-agent-skill
```

Alternatively, you can clone this whole repository and install it however you want.


## Using NucleantUI Pro

The skill is called NucleantUI Pro. In Claude Code you would use this:

> /nucleantui-pro

And in Codex you would use this:

> $nucleantui-pro

In both cases you can provide specific instructions if you want only a partial review. For example, `/nucleantui-pro Check the platform rules` on Claude, or `$nucleantui-pro Focus on performance` in Codex.

You can also trigger the skill using natural language:

> Use the NucleantUI Pro skill to look for performance problems in this project.


## What it checks

| Reference | Covers |
| --- | --- |
| `views.md` | `@View`, initializers and `_viewID`, view structure, missing views, animation |
| `api.md` | SwiftUI API that is missing, renamed, or different in NucleantUI |
| `data.md` | `@Observable` models, `@State`, bindings, identity |
| `navigation.md` | `NavigationStack`, toolbars, popovers, context menus |
| `platforms.md` | cross-platform capability types, per-platform files, Foundation off Darwin |
| `rendering.md` | Skia on Vulkan, `ThorCanvas`, PyShader shaders, textures |
| `design.md` | consistent styling, stable text, mouse and touch, light and dark |
| `resizability.md` | resizable windows and safe areas |
| `accessibility.md` | what applies before NucleantUI has an accessibility API |
| `performance.md`, `performance-plus.md` | cheap views, view inputs, measuring with tracing |
| `swift.md` | modern Swift and Swift concurrency |
| `hygiene.md` | manifests, dependencies, tests, maintainability |
| `framework.md` | working on NucleantUI itself or a library that extends it |


## Contributing

Contributions are welcome, whether that's adding new checks, improving existing ones, or editing this README.

- Keep your Markdown concise. There is a token cost to using skills, particularly with SKILL.md, so please respect the token budgets of users.
- Do not repeat things that LLMs already know. Focus on what NucleantUI does differently from SwiftUI, edge cases, and surprises.
- Check every rule against NucleantUI's sources. A rule must never recommend API that NucleantUI does not have.
- All work must be licensed under the MIT license.


## License

Available under the [MIT License](LICENSE). The original SwiftUI Pro skill is copyright Paul Hudson, also under the MIT License; the NucleantUI adaptation is copyright NucleantUI.
