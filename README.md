# SnapStudio MCP server

Let Claude (or any MCP client) design on your Mac. [SnapStudio](https://getsnapstudio.com) is a native macOS design studio with a built-in MCP server: **87 tools** for templates, brand kits, App Store screenshots, social posts, carousels, short MP4s, on-device upscaling and background removal, and screenshot redaction. Everything renders locally.

> **Requires the SnapStudio app** (macOS 14+, Apple Silicon), from [getsnapstudio.com](https://getsnapstudio.com) ($39.99 once) or the [Mac App Store](https://apps.apple.com/app/id6787644935). The server ships inside the app and updates with it. This repository holds the install docs and a one-click Claude Desktop extension that starts that bundled server; the app itself is not open source.

## What you can ask for

- "Beautify `~/Desktop/dashboard.png` and export it in every App Store screenshot size."
- "Make a 6-slide Instagram carousel from this blog post with my brand kit."
- "Find a product-launch template, fill it with this headline and my logo, then show me a preview."
- "Upscale this photo 4x and remove the background."
- "Find any API keys or emails in this screenshot and black them out before I post it."

## Install

### Claude Desktop (one click)

Download `snapstudio.mcpb` from the [latest release](https://github.com/lukBB/snapstudio-mcp/releases/latest) and double-click it. Claude Desktop installs it as an extension; you can pick the workspace folder in its settings.

### Claude Code

```bash
claude mcp add -s user snapstudio -- /Applications/SnapStudio.app/Contents/MacOS/snapstudio-mcp
```

### Claude Desktop (manual), Cursor, Windsurf and other JSON configs

```json
{
  "mcpServers": {
    "snapstudio": {
      "command": "/Applications/SnapStudio.app/Contents/MacOS/snapstudio-mcp",
      "args": []
    }
  }
}
```

### Codex

```toml
[mcp_servers.snapstudio]
command = "/Applications/SnapStudio.app/Contents/MacOS/snapstudio-mcp"
args = []
```

The server speaks MCP over **stdio**. The app does not need to be open.

## Where files go

By default the agent writes to `~/Library/Application Support/SnapStudio/AgentWorkspace`. Pass `--workspace /path/to/folder` to use another folder. The server reads source images from anywhere you point it, but never writes outside the workspace.

## Privacy

Upscaling, background removal, auto-enhance and sensitive-data detection run on your Mac with Core ML and Vision. No account, no analytics. `article_to_formats` fetches the URL you give it. See the [privacy policy](https://getsnapstudio.com/privacy/).

## Tools

| Tool | What it does |
|---|---|
| `list_documents` | List .snapstudio documents in the workspace with canvas size, layer count, and modification date. |
| `duplicate_document` | Copy an EXISTING document (layers AND image assets) to a new name — the safe way to branch a variant before editing. |
| `duplicate_layer` | Duplicate a layer (groups duplicate their whole subtree) with fresh ids, inserted just above the original. |
| `list_fonts` | List font families available for text layers (installed + app-bundled). |
| `create_document` | Create a new empty document. |
| `list_templates` | List design templates (id, name, category, size) with their fillable slots (text/image, keyed by slot name). |
| `list_palettes` | List the shared color palettes (id, name, family, role colors) — built-ins plus saved brand palettes. |
| `save_palette` | Save a brand palette (edit-and-save-new): start from based_on (any palette) and override any role colors, or specify all roles. |
| `delete_palette` | Delete a saved user palette (built-ins can't be deleted). |
| `list_brand_kits` | List saved brand kits (colors, fonts, logo, URL). |
| `save_brand_kit` | Save a brand kit: colors (from an existing palette or inline), heading/body fonts, URL/handle, and a logo image. |
| `apply_brand_kit` | Apply a brand kit to a document: recolor the background + text (heuristic role mapping), set brand fonts, and place the logo. |
| `generate_social_asset` | ONE CALL from brief to branded asset: template + brand kit + slot content → filled, branded document; optionally export sizes in the same call (preset or variants). |
| `list_assets` | Search the reusable asset library (logos, icons, textures) by name or tag. |
| `save_asset` | Save a reusable image (or an AUDIO music bed via audio_path) to the asset library with a name and tags. |
| `add_asset_to_document` | Place a library asset (by id from list_assets) into a document as an image layer. |
| `create_from_template` | Create a document from a template with its designed placeholder content. |
| `fill_template` | Create a document from a template with slot content applied in one call: texts maps slot name to replacement copy (typography stays as designed; copy longer than the slot box auto-shrinks the font unl |
| `render_template_batch` | Render one PNG per fill from a single template — batch asset generation. |
| `relayout_template` | RE-LAYOUT one template into other formats — the smart alternative to cropping. |
| `article_to_formats` | ARTICLE → CAROUSEL in one call: reads a URL (or html/title+points you supply), plans cover → key points → CTA, re-lays the template out for every requested format, and renders numbered slides into <ou |
| `generate_marketing_set` | Beautify a screenshot (gradient background, rounded corners) and export it in curated social-media sizes (Instagram square/story/portrait, Facebook, X, YouTube, LinkedIn) as PNGs into output_dir. |
| `get_document` | Get the document's canvas + full layer tree (ids, names, types, frames) — the agent's DOM. |
| `add_image_layer` | Add an image layer from a file path (readable anywhere) or base64 data. |
| `add_text_layer` | Add a text layer. |
| `measure_text` | Measure whether text fits a frame BEFORE creating a layer: returns fits true/false, the rendered size, and (when it overflows) the largest font size that fits. |
| `add_shape_layer` | Add a shape layer. |
| `set_layer_transform` | Move/resize/rotate a layer (any omitted field keeps its value). |
| `reorder_layer` | Change a layer's stacking order among its siblings (layers render bottom→top). |
| `add_pattern_layer` | Add a procedural background pattern (grid, dots, diagonal stripes, waves) as an image layer — no external tooling needed. |
| `add_mesh_background` | Add a macOS-wallpaper-style multi-color mesh background (colors anchored around corners, softly blended) as a bottom image layer. |
| `list_components` | List reusable design components (CTA button, slide badge, URL footer, category header, separator, photo overlay, slide counter, stat card, logo lockup) with their text slots — drop them into a documen |
| `add_component` | Insert a reusable component into a document as one group layer, painted from a palette or brand kit so it lands on-brand. |
| `lint_design` | Design QA: flags text overflow, layers mostly off-canvas, empty image slots, low text/background contrast, overlapping text layers (collisions), and text/logo outside the format's safe area (Reels/Sho |
| `score_design` | Design quality score 0–100 with sub-scores (readability, text density, contrast, safe area) — gate export on quality. |
| `render_contact_sheet` | Render up to 20 documents and/or image files side by side in one labeled grid image — review a whole carousel (Instagram allows 20 slides) at a glance, or compare PNGs already on disk without leaving  |
| `set_layer_props` | Edit a layer in place — the one call for everything about an existing layer. |
| `remove_layer` | Remove a layer. |
| `render_preview` | Render the composited canvas as a PNG image — LOOK at your work after editing. |
| `export_image` | Export the composited document to a file in the workspace (png/jpeg/webp/heic/tiff/gif/bmp/pdf). |
| `open_in_app` | Open the document in the SnapStudio app for the human to see. |
| `set_adjustments` | Set photo adjustments on an image layer (only provided fields change; 0 = neutral). |
| `list_filters` | List the built-in filter presets (id, name, category). |
| `apply_filter` | Apply a filter preset to an image layer (replaces its adjustments). |
| `crop_layer` | Crop an image layer to a rect in IMAGE PIXELS, TOP-LEFT origin, y down — the same convention detect_sensitive_data reports and redact_layer's `regions` expects, so a rectangle from either can be passe |
| `set_canvas_size` | Change the canvas size; layers keep their pixel positions relative to the chosen anchor. |
| `upscale_layer` | AI-upscale an image layer's pixels with RealESRGAN (2x or 4x). |
| `remove_background` | Remove the background of an image layer (Vision subject mask). |
| `detect_sensitive_data` | Scan an image layer — or every image layer in the document, if layer_id is omitted — for e-mails, API keys, card numbers, IBANs, phone numbers, IP addresses, tokenised URLs, home paths and faces. |
| `redact_layer` | Permanently paint over (black out, pixelate, or blur) regions of an image layer — the sensitive-data eraser for screenshots. |
| `auto_enhance` | One-click intelligent enhance of an image layer (histogram analysis; replaces pixels). |
| `set_hsl` | Per-color-band HSL grading on an image layer. |
| `set_tone_curve` | Set a tone curve on an image layer (2-5 control points, 0-1 space). |
| `list_stickers` | List built-in vector stickers. |
| `add_sticker` | Add a vector sticker as a shape layer. |
| `list_frames` | List photo frame styles. |
| `apply_frame` | Bake a photo frame around an image layer's pixels. |
| `list_mockups` | List device mockup frames (Mac, iPhone, iPad, browser, watch). |
| `apply_mockup` | Frame a TOP-LEVEL image layer inside a device mockup. |
| `list_beautify_styles` | List screenshot beautifier styles. |
| `beautify_layer` | Screenshot beautifier: gradient backdrop, padding, rounded corners and shadow around a TOP-LEVEL image layer. |
| `list_collage_templates` | List collage layouts. |
| `create_collage` | Create a collage document from image files (one per slot, in order). |
| `save_as_template` | Save the document's layer structure as a reusable user template (appears in the app's gallery too). |
| `delete_template` | Permanently delete ONE user template by its exact id (use list_templates or inspect_template to find it — a name is not accepted here because a name can match several templates, and a delete must neve |
| `preview_template` | Render a template's preview image (built-in or user; embedded logos/photos included, empty image slots shown as gray placeholders) — LOOK at a template before choosing it. |
| `generate_aso_pack` | COMPLETE store listings in one call: renders the built-in aso_* families (hero, features, CTA) with your app screenshots and copy for every language x platform — iPhone, iPad, Google Play AND the Mac  |
| `inspect_template` | Structured JSON for one template: canvas, slots (name/role/default), full layer tree with frames/fonts/kinds, folder/family, palette-swappability — the machine-readable alternative to list_templates p |
| `diagnostics` | Server healthcheck / restart status: an ok line, the git commit this binary was built from (and whether the working tree was dirty at build time), whether a NEWER build has since been deployed to bin/ |
| `render_carousel` | One-call social carousel (Instagram max 20 slides; YouTube Shorts max 10; sizes per platform: 1080x1080, 1080x1920, 960x1200, 1080x1350). |
| `save_video_template` | Save a reusable VIDEO template (CapCut-style): scenes reference layer templates (with slots) plus per-layer animations, holds, transition and preset. |
| `list_video_templates` | List saved video templates with their scenes, layer templates, and fillable slots. |
| `render_video_template` | Render a saved video template into a finished MP4: per-scene slot fills (texts/image_paths like fill_template, plus clip_paths for VIDEO in image slots), optional palette/music/bpm override. |
| `delete_video_template` | Delete a saved video template by id or name. |
| `export_animated_post` | Animated social MP4 where each document's LAYERS animate in (slide/fade/pop, staggered bottom-to-top), hold fully visible, then cut/crossfade to the next document — motion-design posts straight from d |
| `export_slideshow` | Turn images or documents into a social MP4 slideshow (30 fps): per-slide duration, hard-cut or crossfade, optional Ken Burns pan-zoom and background music. |
| `export_templates` | Export user templates to a portable directory (one sidecar bundle per template, image assets included) for backup or moving to another machine — re-importable with import_templates. |
| `import_templates` | Import templates from a directory made by export_templates (or a copy of another machine's Templates dir). |
| `export_platform_pack` | Export the composited document at every size in a platform pack (App Store screenshots, Google Play incl. |
| `set_layer_mask` | Mask a layer to a rectangle or ellipse region (layer-local coordinates, top-down). |
| `clear_layer_mask` | Remove a layer's mask. |
| `add_drawing_layer` | Add a freehand drawing layer from polyline strokes (layer-local, top-down). |
| `clone_heal` | Clone or heal along a path on an image layer (image-pixel coordinates, top-down). |
| `apply_perspective` | Warp (apply) or de-keystone (correct) an image layer's pixels by four corners (image pixels, top-down). |
| `import_svg` | Import an SVG file as a vector layer group (optionally placed into a frame). |
| `export_svg` | Export the document as SVG (vector layers stay vector; trace_images vectorizes photos). |
| `batch_process` | Run one operation over every image in a folder (max 500). |

## Links

- Website and setup guide: https://getsnapstudio.com/mcp/
- Support: support@getsnapstudio.com
- Changelog: https://getsnapstudio.com/changelog/

The launcher script and docs in this repository are MIT licensed. SnapStudio itself is proprietary software.
