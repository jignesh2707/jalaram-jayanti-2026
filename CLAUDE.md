# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file, mobile-first web app (`index.html`) that lets people make a personalized Jalaram Jayanti 2026 (16/11/2026) invitation poster: pick a photo, position it in the frame, type their name, then download or share the resulting JPEG. All UI text is Gujarati (`lang="gu"`).

There is no build, package manager, linter or test suite. Open `index.html` in a browser (or serve the folder, e.g. `python3 -m http.server`) to run it. Test on a phone-sized viewport: touch drag/pinch and `navigator.share` are core paths.

## Working with `index.html`

- **Line 169 (`var TPL`) is a huge base64 WebP data URI (`TPL`)** — the poster template, which has a transparent hole where the photo shows through. Never `Read`/`cat` the whole file or that line; use offsets/ranges or `cut -c1-200`. To replace the template, swap the data URI and re-check the geometry constants below.
- Everything (CSS, markup, JS) is inline; JS is one ES5-style IIFE (`var`, no modules, no transpiling). Keep it that way for old mobile browsers. Only external dependency is Google Fonts (Baloo Bhai 2, Hind Vadodara).
- Colors are CSS tokens on `:root` with dark-mode overrides under `prefers-color-scheme` / `data-theme`.

## Architecture

**Poster coordinate space.** Everything is drawn in the template's native 937×1678 pixel space, defined by constants at the top of the script:
- `HOLE` — rectangle behind the template where the photo is clipped/drawn
- `OVAL` — the visible photo frame; base photo scale fits the image into it (`baseScale`), "fill" mode uses `coverZoom`
- `PLATE` — name plate; `drawName` shrinks the font until the text fits `maxW`
- `CROP` — the region of the poster shown in the step 2/3 editor canvases

These must stay consistent with the template image.

**Layering (`drawScene`).** White fill + blurred low-res copy of the photo (`makeBlur`) in `HOLE` → user photo at `baseScale()*zoom` offset by `offX/offY` → template image on top → name text. The same function renders the step-1 preview, the editor canvases (`drawEditor` applies a `CROP`-based transform scaled for devicePixelRatio) and the final output canvas (`drawFull`).

**Step flow.** Four `<section class="step">` elements (`s1`–`s4`) toggled by `go(n)`; `updateNext()` gates the Next button (step 2 needs a photo, step 3 needs a name). Step 4 runs `buildFinal()`, which renders to an offscreen 937×1678 canvas and produces a JPEG blob plus a data-URL `<img>` (data URL deliberately, since some hosts block `blob:` images; falls back to inserting the canvas itself).

**Photo loading robustness.** `loadFile` tries object URL → FileReader data URL → `createImageBitmap`, then downscales to max 1600px (`toCanvas`) to avoid mobile memory issues. The file input is triggered via a `<label for="file">` so phones open the native gallery; both `change` and `input` events are handled with a debounce flag.

**Download/share.** `doDownload` prefers the `window.claude.use('downloads')` runtime capability (the page is published as a claude.ai Artifact) and falls back to an `<a download>` link. `doShare` uses the Web Share API with files, falling back to download plus instructions.
