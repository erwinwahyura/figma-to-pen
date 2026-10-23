# figma-to-pen

Convert Figma designs into [pen.dev](https://pen.dev) `.pen` documents (schema 2.21). Everything runs in the browser, and no file leaves your machine.

It carries across frames, auto-layout, components and instances (with overrides), text, shapes, vector paths, gradients, image fills, effects and variables (with modes as themes). After each conversion, a report lists anything that couldn't be converted exactly.

## Usage

Open `index.html` in a browser, or host the folder as a static site. Then either:

- drop a `.fig` file or a `.json` file anywhere on the page,
- paste JSON into the left pane, or
- click **Load sample** to see an example.

Click **Convert to .pen**, then **Download** or **Copy .pen JSON**. You can choose which pages to convert and turn off hidden layers, components or variable binding.

## Input formats

| Source | How to get it | Notes |
| --- | --- | --- |
| Figma REST API | `GET /v1/files/:key?geometry=paths` | Most reliable. `geometry=paths` is needed for icons and vectors. |
| REST variables | `GET /v1/files/:key/variables/local` | Optional. Paste it under **Variables JSON**. Needs an Enterprise plan. |
| Plugin export | `figma-plugin.zip` (see below) | Works on any plan. Includes variables and the components the selection uses. |
| `.fig` file | Figma → File → Save local copy | Best effort. This is Figma's undocumented internal format. Variables aren't read. |

```sh
curl -H "X-Figma-Token: $FIGMA_TOKEN" \
  "https://api.figma.com/v1/files/FILE_KEY?geometry=paths" -o figma.json
curl -H "X-Figma-Token: $FIGMA_TOKEN" \
  "https://api.figma.com/v1/files/FILE_KEY/variables/local" -o variables.json
```

### Images

For REST JSON, image fills point to `./images/<imageRef>.png`. Download the image files with `GET /v1/files/:key/images` and put them in an `images/` folder next to the `.pen` file. For a `.fig` file, **Download** produces a zip that already contains `design.pen` and its images.

## Figma plugin

`figma-plugin.zip` contains a small plugin called **Export for pen.dev**. It exports the current selection, the local components that selection uses and the file's local variables as one JSON document.

1. Unzip it.
2. In the Figma desktop app, go to **Plugins → Development → Import plugin from manifest…** and pick `manifest.json`.
3. Select one or more frames and run the plugin.
4. Click **Copy JSON** and paste the result into the converter.

The plugin has no network access.

## Known limitations

The conversion report flags each of these when it happens:

- Dashed strokes become solid. Shadow spread is dropped.
- Wrapping and grid auto-layout aren't supported.
- Text with mixed styles is flattened to its base style.
- Tiled image fills become `cover`. Diamond gradients become radial.
- Rotation is only kept for `.fig` input. Layers that are both rotated and flipped aren't mirrored.
- Masks clip to the mask's bounding box. A vector mask over a single fill, which is how most icons are built, keeps its exact shape.
- Truncated text is clipped to its box, without the ellipsis.
- FigJam and whiteboard layers (stickies, connectors, tables and so on) are skipped.

## Files

- `index.html`: the converter (UI, `.fig` decoder and conversion code in one file)
- `figma-plugin.zip`: the Figma export plugin
- `_headers`: security headers and CSP for static hosts such as Cloudflare Pages or Netlify
