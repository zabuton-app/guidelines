# 04 Theme

## Base16 Is the Source of Truth for the Color System

Every theme is defined as a 16-color [base16](https://github.com/chriskempson/base16) scheme and applied to the UI through semantic tokens. Components never hold raw color values; they must always reference semantic tokens.

### Three-Stage Bridge

```text
base16 scheme → --c-* (runtime CSS variables) → --color-* (Tailwind) → components
```

1. `SEMANTIC_MAP` in `src/themes/base16.ts` maps base16 → semantic names:
   `bg`(base00), `surface`(01), `overlay`/`border`(02), `muted`(03), `secondary-fg`(04), `fg`(05), `bright-fg`(06), `highlight`(07), `error`(08), `warn`(09), `accent2`(0A), `success`(0B), `info`(0C), `primary`(0D), `secondary-accent`(0E), `special`(0F)
2. `schemeToCssVars()` generates `--c-*` as a `[data-theme="<id>"]` block, and `ThemeProvider` injects it into a single `<style>` element
3. `@theme inline` in `src/styles.css` bridges it as `--color-*: var(--c-*)`, to be used as utilities such as `bg-bg` / `text-fg` / `border-border`. shadcn-compatible aliases (`--color-background`, `--color-primary-foreground`, etc.) are defined in the same block

## Required Theme Families

Every app supports the following 12 families as the main themes. Each family must have a light / dark pair (24 schemes in total).

| Family | Notes |
| --- | --- |
| Gruvbox | Default theme (`gruvbox-dark`) |
| Solarized | |
| Nord | |
| Catppuccin | |
| Tokyo Night | |
| Ayu | |
| Material | |
| Monokai | |
| GitHub | |
| One | |
| Rosé Pine | |
| Tomorrow | |

## Sharing Theme Definitions

- `src/themes/base16.ts`, `src/themes/schemes.ts`, and `src/themes/ThemeProvider.tsx` are series-wide shared code, kept in sync across apps by copying (no differences allowed; only the `<app>.theme` prefix of the localStorage key and the `<style>` element id may be replaced with the app name)
- A new theme is added to meguri's `schemes.ts` first and then rolled out to every app

## Switching and Persistence

- Dark mode is implemented by switching the `<html data-theme="<id>">` attribute, not by `prefers-color-scheme`. Each scheme has `appearance: "light" | "dark"`, and `setMode` switches between the pair within a family
- Persistence uses `localStorage` (key: `<app>.theme`, e.g. `meguri.theme`)
- To prevent FOUC, load `public/theme-boot.js` in the `<head>` of `index.html` and apply `data-theme` before the first paint

## Theme Preview

The theme cards on the settings screen show swatches of **six semantic tokens** (`bg` / `border` / `muted` / `primary` / `accent2` / `error`) passed through `deriveTokens()`.

Derived values are used instead of raw palette slots because the swatches must match the colors actually used in the UI. Some upstream base16 schemes have non-monotonic ramps or repurpose slots as accents, so lining up the raw values would not let users judge, before switching, the chrome colors (hairlines, secondary text) that become invisible in some themes.

The source of truth for the implementation is `PREVIEW_TOKENS` in meguri's [`src/routes/Settings/index.tsx`](https://github.com/zabuton-app/meguri/blob/main/src/routes/Settings/index.tsx).
