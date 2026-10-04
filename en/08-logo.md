# 08 Logo

Logos in the zabuton series use [meguri's logo](https://github.com/zabuton-app/meguri/tree/main/logo) (`meguri-icon-1f.svg`) as the base design.

## Composition Rules

- **The motif is the single kanji of the app's display name** (see [00 Brand and Naming](00-brand.md))
- **The glyph color is the series-wide kinari (off-white) `#f6efe0`**. Only when the background is dark and does not provide enough contrast with this kinari may an app-specific light color be used for the glyph and the inner border rule. When adopted, list it alongside in the color table
- **The background is a gradient of the app-specific color**. The app's individuality is expressed through this background color. When the color is so dark that the gradient difference is not visible, a solid color is fine
- The glyph (text element) **must be converted to paths (outlined)** in distributed artifacts. Do not depend on the font environment

## File Layout

Put the following in `<repository root>/logo/`.

- `<app>-icon.svg` — The editable master (may contain text elements)
- `<app>-icon.paths.svg` — Converted to paths. Exports and distribution are based on this file

## Export Sizes

meguri's established set is the standard. Generation is automated by the `icon-gen` skill (`.claude/skills/icon-gen/generate-icons.sh`), which produces the PNGs below from the logo SVG in one go, along with `build/icon.{png,ico,icns}` for electron-builder and the appx tiles for the Microsoft Store.

- **App icons**: `app-32.png` / `app-64.png` / `app-128.png` / `app-256.png` / `app-512.png`, and `appicon-1024.png`
- **Tray icons** (required for every app; see [01 Layout](01-layout.md)): `tray-16.png` / `tray-32.png` / `tray-64.png` / `tray-256.png`

## Colors per App

| App | Color name | Gradient |
| --- | ---------- | -------- |
| 巡 (meguri) | Shu (vermilion) | `#bd4028` → `#93301c` |
| 刻 (kizami) | Tomato | `#ff6b57` → `#e0432e` |

When assigning a color to a new app, look at this whole table so that apps with similar hues do not end up side by side.

## TODO: Undecided Conventions

- Quantifying the safe area (the margin ratio between the glyph and the outer edge)
- Handling of background corner radius and transparency (how to deal with per-OS mask differences)
