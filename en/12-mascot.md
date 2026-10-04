# 12 Mascot

The zabuton series adopted a mascot modeled on a navy zabuton cushion as its official character (2026-10). The tassels at its four corners serve as its arms and legs, and it is treated as the face of the whole series.

## Role

- **The mascot is the character of the series (the zabuton brand)** and does not belong to any particular app
- **An app's logo remains the single kanji of its display name** (see [08 Logo](08-logo.md)). Do not replace app icons or tray icons with the mascot
- The pixel-art zabuton that appears in meguri's pet feature (`src/components/pet/`) has the same design as this mascot

## Where the Assets Live

The originals are maintained in the [`zabuton-app/mascot`](https://github.com/zabuton-app/mascot) repository. That repository's README is the source of truth for file naming rules and animation frame composition.

| Kind | Contents | Format |
| ---- | -------- | ------ |
| Vector version | `mascot-a` (pose with both hands raised), `mascot-c-no-enso` (pose with hands down and blushing cheeks) | SVG, 2048px PNG |
| Pixel-art version | Sprites on a 24×20 grid. 16 animations | Per-frame SVG and PNG, looping GIF |

When used in an app or a site, the assets are **shared by copying**, like other shared resources. Do not introduce differences after copying; make fixes in the `mascot` repository and then pull them in again.

## Choosing a Variant by Background

Because the body is navy, the outline sinks into dark backgrounds as is. Every asset has a regular version and an `-on-dark` version, so choose according to the background.

- **Light background**: the regular version (the vector version has no outline; the pixel-art version has a sumi-ink `#0b1520` outline)
- **Dark background**: the `-on-dark` version (with a kinari `#f6efe0` outline)
- Where the background color cannot be specified (OGP images, store listing images, etc.), use the `-square` version (kinari `#f6efe0`) or the `-on-dark-square` version (`#1d2021`), which include a background

## Colors

| Use | Color |
| --- | ----- |
| Body | `#132537` |
| Body shadow | `#0b1826` |
| Highlight | `#243a55` |
| Trim and tassel knots | `#d9b062` |
| Tassels | `#b27f31` |
| Eyes, mouth, and the outline of the `-on-dark` version | `#f6efe0` (the same kinari as the logo glyph color) |

Do not create recolored variations (such as repainting it in an app-specific color).

## TODO: Undecided Conventions

- The mascot's name
- Whether `mascot-a` or `mascot-c-no-enso` is the default pose
- Where to draw the line on usage (rules for appearing in the About section, landing pages, store listing images, etc.)
- Terms of use for third parties (the license of the `mascot` repository)
