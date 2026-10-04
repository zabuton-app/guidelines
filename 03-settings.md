# 03 Settings Screen

Implement the settings screen as a routed modal ([02 Navigation](02-navigation.md)). The implementation lives in `src/routes/Settings/`, split into `index.tsx` (screen assembly) + `SettingsModal.tsx` (frame) + per-section files (`AboutSection.tsx`, etc.). When a section has state or event subscriptions, always put it in a separate file and leave only the row pattern markup in `index.tsx`.

## Modal Dimensions

- Overlay: `fixed inset-0 z-50 flex items-center justify-center bg-black/70 p-4 backdrop-blur-sm`
- Panel: `relative flex h-[85vh] w-full max-w-xl flex-col overflow-hidden rounded-xl border border-border bg-bg shadow-2xl`
- Header: title + a close button at the right end (`X` icon, with "Close (Esc)" as its `title`). Fixed with `border-b border-border bg-bg px-4 py-2.5`
- Body: `mx-auto flex max-w-2xl flex-col gap-8` inside a `ScrollArea` (`min-h-0 flex-1`, `viewportClassName="p-4 pb-6"`)

## Section Order

The default is a single scrolling stack of sections. The order is fixed as follows.

1. Language
2. App-specific settings (meguri: Scenes / Keybinding, etc.)
3. Appearance (light / dark)
4. Theme (theme family selection)
5. Update (only for apps with automatic update checks)
6. Support (donation link, optional)
7. About (required. See [06 About and Licensing](06-about-and-licensing.md))

### Exception: Tabbed Layout

An app whose settings have grown to the point where a single scroll no longer gets users to the item they want may split them into tabs by category. meguri is such an app, implemented as `src/routes/Settings/SettingsTabs.tsx` (tabs introduced on 2026-08-29; this section added on 2026-09-23).

Even when splitting into tabs, keep the following.

- Do not change the modal dimensions, the row pattern, or the control selection criteria described in this chapter
- Assign sections to categories without breaking the default order above within each tab. Put About at the end of the last tab
- Do not persist the selected tab; start from the first tab every time the modal opens
- A new app starts with a single scroll. Do not split into tabs from the beginning

## Row Pattern

Every section follows the same row pattern: label + description on the left, control on the right.

```tsx
<section className="flex items-center justify-between gap-3 rounded-md border border-border bg-surface px-4 py-3">
  <div className="flex flex-col">
    <span className="text-sm font-semibold text-bright-fg">{/* label */}</span>
    <span className="text-xs text-muted">{/* description */}</span>
  </div>
  {/* control */}
</section>
```

## Control Selection Criteria

- **A list of choices** (language, presets, etc.): the Radix-based `Select` (`@/components/ui/select`)
- **A binary choice** (light / dark): a segmented toggle. Selected is `bg-primary text-primary-foreground`, unselected is `bg-bg text-muted hover:text-fg`. With icons (`Sun` / `Moon`)
- **Theme family**: a card grid (`grid grid-cols-1 gap-2 sm:grid-cols-2`). Each card has six color swatches (semantic tokens passed through `deriveTokens()`; see the theme preview in [04 Theme](04-theme.md)) + the family name, and the selected card gets `border-primary ring-1 ring-primary` + a `Check` icon
- **Free-form multi-line text** (meguri: AI tag vocabulary): a full-width `textarea` with `font-mono text-xs`, `resize-y`, `spellCheck={false}`, and an `aria-label` with the same text as the label
- **A continuous value** (meguri: AI threshold): a full-width `input type="range"`. Show the current value at the right end of the label row in `font-mono`. A native range already has `role="slider"` and arrow key handling, so do not build a custom one
- **A list of items of the same kind** (meguri: AI model list): a single-column list. Each row is `rounded-md border border-border bg-bg px-3 py-2`, with the name + secondary information on the left and actions on the right. The selected row gets a `Check` icon + a "selected" label

### Exception to the Row Pattern

When a control needs the full width (`textarea` / `range` / list), it may be stacked vertically with the label + description on top and the control below. Even then, the label + description markup stays identical to the row pattern.

### When Settings Are Saved

Settings take effect immediately as a rule. Only for settings whose application involves heavy recomputation (meguri: applying a change to the AI tag vocabulary or threshold requires re-tagging the entire library) do you place an explicit save button and show a warning that the change has not been applied yet.
