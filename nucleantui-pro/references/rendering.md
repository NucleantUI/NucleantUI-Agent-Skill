# Rendering

## Skia on Vulkan, always

- Skia draws all of NucleantUI: views, the window canvas, `.drawingGroup()`, `.shader` layers, and all text, measured, loaded, and drawn. The only exception is what a shader draws itself.
- NucleantUI is a Vulkan-based UI with no CPU fallback. Everything renders through Vulkan, snapshots and `renderImage` included. Never write a CPU rendering path – a Skia raster surface, ThorVG's software canvas, or a pixel loop that draws interface – not even as a fallback. A system without Vulkan (a GPU, MoltenVK, or a software Vulkan driver) does not run NucleantUI.
- There is one renderer and no mode switch. Never add one, or suggest comparing output against another backend.
- To get a view's pixels, use `renderImage(size:scale:)` or `.image(size:scale:)`: Skia draws it on a windowless Vulkan device and returns a `RasterImage`.
- Render nodes are automatic: a view that changes gets an image of its own, repainted only when its content changes. Use `.drawingGroup()` only for a few large subtrees that should be one image whatever they read, not for every row; each group owns a Skia canvas.


## `ThorCanvas`

- ThorVG is used by the `ThorCanvas` view (and `ThorCanvasRender`) alone, on its GPU canvas. Never draw with ThorVG anywhere else, and never use its software canvas.
- Give each ThorVG paint a small `final class` of its own conforming to `ThorShape`, owning its paint, rather than using `TCShape`. Create paints in `onInit`, which runs again for a fresh node, and keep them.
- Mutate an existing paint before rebuilding it. Move or resize with `translate` / `scale`, and recolor with `set_fill_color` / `set_stroke_color`. Clear and re-append a path only when the path itself changes, or the shape does not exist yet.
- `renderer` runs with its reads tracked like a `body`, so it re-runs when something it read changes. Sizes there are canvas pixels; `context.scale` is pixels per point.


## Shaders

- Write shaders in [PyShader](https://github.com/NucleantUI/PyShader), Python syntax compiled straight to SPIR-V. Compile them at build time with its build plugin, listing each `.py` in a `pyshader.toml`, or at run time with `PyShaderFunction` from NucleantUI_PyShader. Run a compiled module with `SpirvFunction`, whose input is listed in a builder block or is a `ShaderInput` model that packs itself. Do not write `ShaderFunction` source strings for new shaders.
- `Shader`, `.shader(_:)`, and a `RenderTexture`'s `shader(_:)` are generic over `ShaderProtocol`. Keep them generic; do not erase shader types.
- The set of argument names and kinds is part of the compiled pipeline. Keep it stable and vary the values.
- A shader that reads neither the clock nor the pointer is dispatched once, and again only when its input changes; one that does runs every frame. Do not read the clock in a shader that does not animate.
- Shader views composite as rectangles and are clipped by their containers; `.cornerRadius()` does not round them. To round a `.shader` effect, apply `.cornerRadius()` inside it, so the rounding is in the texture.
- Effects do not nest: a `Shader` view or a second `.shader` inside one is composited over the effect's output, not through it.
- With `backdrop: true`, what was painted under the view is part of the texture. Put a label meant to sit on such a panel inside the effect, not on top of it.


## Textures

- A `TextureView`'s source renders at the texture's own `pixelWidth` × `pixelHeight`, which follow the view's size and scale. Never render at a native size and stretch the result with `.scaleEffect()`; a producer that cannot render at an arbitrary size scales on its own side before writing.
