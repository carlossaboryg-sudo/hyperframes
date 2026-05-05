# Session Memory

## User Profile

- **Language**: Spanish
- **Environment**: WSL Ubuntu on Windows (UNC path issues with CMD — use `C:\Users\carli\AppData\Local\Temp\opencode\` for rendering)
- **Bun**: NOT installed. Use `npx hyperframes` (published version) for rendering. To build locally: needs `bun` installed in WSL first.
- **Node**: Available at `C:\Program Files\nodejs\node.exe` (v24)

## Brand Reference

- **Lumière du Nord** — Official Canadian distributor of Sandstone Scandinavian Cosmetics
  - Website: www.lumieredunord.ca
- **Sandstone Scandinavian** — Denmark-based cosmetics manufacturer and partner
  - Website: www.sandstonescandinavia.com

Always use these references when creating videos: Lumière du Nord is the distributor (Canada), Sandstone Scandinavian is the manufacturer (Denmark).

## Project Focus

Building luxury brand video reels with HyperFrames. Current brands: **Lumière du Nord** / **Sandstone Scandinavia**.

## Templates Created

### `templates/sandstone-reel/`
- 1080x1920 (9:16 vertical), 30fps, 8 scenes (cover + 6 video + closing)
- GSAP timeline with opacity+scale crossfade transitions (NO blur — fails in BeginFrame/SwiftShader mode)
- Turquoise pill button (#7ec8c8 → #6cb8b8) for closing CTA URL
- Cover: logo only (no "presents" text), animated with `tl.set({y:30, opacity:0, scale:0.95})` + `tl.to({y:0, scale:1, opacity:1})`
- Variable markers: `<!-- TEMPLATE: -->` comments in HTML

## Rendering Preferences

- **Transitions**: `opacity + scale` crossfade. NOT `filter: blur()` (doesn't render in BeginFrame mode with SwiftShader).
- **Quality**: `--quality standard` for final, `--quality draft` for iteration
- **FPS**: 30
- **Encoding**: GPU auto-detect preferred. Streaming encode enabled for performance.
- **Output**: MP4

## OpenCode Configuration (2026-05-05)

**Config files created:**
- `opencode.json` — model (`opencode/deepseek-v4-pro`), MCP Context7, formatter (oxfmt), compaction, permissions
- `.opencode/commands/` — 5 custom slash commands for HyperFrames workflow
- `.opencode/agents/` — 3 specialized subagents (hf-lint, hf-validate, hf-review)

**MCP servers:**
- `context7` — live docs search (GSAP, FFmpeg, Puppeteer APIs)

**Available slash commands:**
| Command | Action |
|---------|--------|
| `/lint-hf [dir]` | Lint composition |
| `/validate-hf [dir]` | Validate in Chrome |
| `/render-draft [dir]` | Draft render |
| `/render-final [dir]` | Lint+validate+render high |
| `/doctor` | Environment check |

## Known Issues

1. **video8.mp4**: Has sparse keyframes (max interval 5s). Needs re-encode: `ffmpeg -i video8.mp4 -c:v libx264 -r 30 -g 30 -keyint_min 30 -movflags +faststart output.mp4`
2. **duplicate_media_discovery_risk**: Lint warning on duplicate media entries — safe to ignore if intentional.

## Efficiency Improvements Applied (source code, not yet built)

| # | File | Change |
|---|------|--------|
| 1 | `packages/engine/src/config.ts` | `enableStreamingEncode: true`, `enableChunkedEncode: true`, `gpuAutoDetect: true` |
| 2 | `packages/engine/src/config.ts` | Added `seekStep`, `seekOffsetFraction`, `seekMode` typed fields |
| 3 | `packages/engine/src/services/chunkEncoder.ts` | GPU auto-detect without explicit `--gpu` flag |
| 4 | `packages/engine/src/services/videoFrameInjector.ts` | Cache `activeIdSet` between frames to reduce allocations |
| 5 | `packages/engine/src/services/frameCapture.ts` | `pollPageExpression` → `page.waitForFunction` in screenshot mode |
| 6 | `packages/engine/src/services/parallelCoordinator.ts` | Running counter instead of per-frame Map+reduce |
| 7 | `packages/producer/src/perf-gate.ts` | Memory regression gate (RSS + heap) |
| 8 | `packages/cli/tsup.config.ts` | `minify: true`, `treeshake: true` |
| 9 | `packages/producer/build.mjs` | `minify: true` for both entry points |

## Pending Improvements

- `sharp` integration for HDR compositing (5-20x speedup)
- Double-buffering in streaming encoder (eliminate 37MB memcpy per HDR frame)
- `Uint16Array` migration in alphaBlit (2x pixel ops)
- CDP round-trip merging in HDR hot loop
- Install `bun` in WSL to build with local optimizations

## Project Structure Convention

All HyperFrames composition projects live under `projects/`. Each project has its own `assets/` folder for project-specific media. Shared media (logos, fonts, etc.) goes in the root `assets/` folder.

```
hyperframes/
  assets/          ← shared logos, images, fonts used across projects
  projects/        ← all composition projects
    <project-name>/
      index.html
      assets/      ← project-specific media (video, images, audio)
      renders/     ← rendered MP4 output
```

**Key rules:**
- New projects: always create them inside `projects/`
- Shared logos/images: place in root `assets/`, reference from HTML as `../assets/filename.ext`
- Project-specific media: place in `projects/<name>/assets/`, reference as `assets/filename.ext`
- Root `renders/` is NOT used — each project has its own `renders/` folder

## Projects

### `projects/lipglace-promo/` — Lipglace Innocent Promo
- Brand: Lumière du Nord / Sandstone Scandinavia
- Product: lipglace-innocent

### `projects/lluvia-roja-promo/` — Lluvia Roja Promo
- Product: Intense Care Lipstick

### `projects/Video Marce/` — Brand Reel (Sandstone Scandinavia)
- `index.html`: 1080x1920, 36.2s, 30fps, 8 scenes, videos + GSAP overlay crossfades
- Rendered: `projects/Video Marce/renders/` (16.2 MB)
- Cover: logo only, no "presents"
- Closing: turquoise pill for URL

### `projects/Sandstone History/` — History Reel (Sandstone Scandinavia)
- `index.html`: 1080x1920, 55s, 30fps, 6 scenes, static images + GSAP text overlays
- Rendered: `projects/Sandstone History/renders/` (3.7 MB)
- Fonts: Cormorant Garamond + Jost (68 Google Font faces cached)
- Extracted from Claude Design bundle via `extract.cjs`
- Scene 3: text-only quote scene (no image)
- Font sizes: headlines 64-92px, body 44px, quotes 72px, pill 36px
- Lint warnings: `class="clip"` on imgs (safe — GSAP handles visibility), duplicate logo (used in cover+closing)

## Render Command Reference

```bash
# Windows temp workaround (UNC path bypass):
Copy-Item -Recurse "\\wsl.localhost\Ubuntu\home\carli\hyperframes\projects\<project>" "C:\Users\carli\AppData\Local\Temp\opencode\<project>" -Force
npx hyperframes render "." --quality draft --fps 30   # from temp dir

# Copy result back to WSL:
wsl -d Ubuntu -- bash -c "cp '/mnt/c/Users/carli/AppData/Local/Temp/opencode/<project>/renders/<file>.mp4' '/home/carli/hyperframes/projects/<project>/renders/'"

# Standard (with bun):
node ~/hyperframes/packages/cli/dist/cli.js render . --quality standard --fps 30 --gpu
npx hyperframes benchmark
npx hyperframes lint

# Claude Design bundle extraction:
# 1. Write extract.cjs script
# 2. Run: wsl -d Ubuntu -- bash -l -c "cd /path && '/mnt/c/Program Files/nodejs/node.exe' extract.cjs"
# 3. Convert extracted template to HF format (GSAP timeline, data-* attrs, fixed canvas)
```
