# 05 Components

## Shared UI Components (shadcn/ui Pattern)

- Put in-house implementations of the shadcn/ui pattern (Radix + CVA) in `src/components/ui/`: `button` / `dialog` / `select` / `dropdown-menu` / `scroll-area` / `input` / `skeleton`, etc.
- Join classes with `cn()` (`clsx` + `tailwind-merge`, `src/lib/utils.ts`)
- Standardize on lucide-react for icons, sonner for toasts, and dnd-kit for reordering
- **For confirmation dialogs, use the in-app modal `ConfirmDialog` (`useConfirm()` returns a `Promise<boolean>`)**. Native dialogs such as `window.confirm` / `window.alert` are prohibited (do not create a path that opens any window other than the main window). Put `ConfirmProvider` at the app root

### Scrollbars

- **Every visible scrollbar is the shared `ScrollArea` (`src/components/ui/scroll-area.tsx`)**. Do not leave a platform scrollbar on screen: an element with `overflow-y-auto` / `overflow-auto` draws the OS one, which ignores the theme and differs per platform
- Size the root, not the viewport: `min-h-0 flex-1` inside a flex column (a modal body, a tab panel). Padding for the content goes in `viewportClassName`
- A region capped by `max-h-*` puts the cap in `viewportClassName`. The root clips without scrolling when the cap is on it
- When the content must fill the viewport even while it is short (a pane border that runs to the bottom, a footer pinned with `sticky bottom-0`), pass `fillViewport` and give the content `flex-1`. Radix wraps the content in a `display: table` box that is only as tall as the content, so a percentage `min-h-full` on the content itself does not work; `fillViewport` restyles that box, and knowledge of it stays inside `scroll-area.tsx` — do not reach for it with `[&>div]:` selectors from a screen
- Small incidental overflows that need no visible bar (a tab strip or a segmented control that may not fit a narrow window) keep `overflow-x-auto` and hide the bar with the `no-scrollbar` class from `src/styles.css`
- A list virtualized with `@tanstack/react-virtual` passes `viewportRef` to `ScrollArea` and uses it as `getScrollElement`

### Copy-Based Sharing Rules

- Shared components are kept in sync across apps by copying. **Do not introduce app-specific differences after copying** (avoid formatting-only differences too)
- Improvements go into meguri first and are then rolled out to the other apps
- Keep the Button variants `default / secondary / outline / ghost / destructive` × sizes `default(h-9) / sm(h-8) / compact(h-7) / lg(h-10) / icon(h-9 w-9)`

## Design Tokens

The scales every part takes its values from, defined in meguri's `src/styles.css` and described in [meguri's docs/renderer.md](https://github.com/zabuton-app/meguri/blob/main/docs/renderer.md#design-tokens) ([meguri#584](https://github.com/zabuton-app/meguri/issues/584)). A value outside them needs a reason.

- **Corner radii, by role**: `rounded-panel` (16px, the content panel set in the frame), `rounded-window` (12px, modals and dialogs), `rounded-card` (8px, cards, panels, list rows), `rounded-control` (6px, buttons, fields, menus, popovers, badges, toasts), `rounded-item` (4px, menu items, small chips, thumbnails inside a row or a card), `rounded-full` (pills). A part inside another gets a smaller radius than its container. `cn()` is extended with these names so a radius passed in `className` replaces a component's own
- **Spacing**: Tailwind's steps 0.5, 1, 1.5, 2, 3, 4, 6 and 8 (2 to 32px)
- **Control heights**: 24px (`h-6`, icon buttons on cards and rows), 28px (`h-7`, `Button size="compact"`, dense rows and toolbars in a window), 32px (`h-8`, `Button size="sm"`, `Input`, `Select`)
- **Font sizes**: `text-2xs` (11px, badges, counts, secondary lines; no line height of its own), `text-xs` (most UI text), `text-sm` (body text, menu items, fields), `text-base` (window titles)
- **Borders**: 1px; 2px only for focus and selection rings. `border-border` for dividers and containers, `border-border-strong` for the edge of a control

### States

- **On**: a control that is on (a toggle, a selected segment, a pressed filter, the selected rail item) is tinted with `bg-primary/15`, never a solid fill. Its outline stays `border-border`. A favorite keeps its meaning in red (`bg-error/15 text-error`)
- **Hover**: a bare button or row takes `bg-fg/10`; a selected one `bg-primary/25`
- **Selection and focus** in a grid or a list: a tint marks what is selected, and a 2px `ring-primary` marks where the focus is, so the two show together. Selected items carry `data-selected` and the focused one `aria-current="true"`, for CSS themes

### Windows and Dialogs

- **No boxes inside a window**: sections in a full-size window (settings, file details) are separated by a rule (`border-b` / `border-t border-border`), not each put in a box. Boxes are kept only for what holds content: a list, a notice, a set of choices
- **Small dialogs** (the shared `Dialog`, `ConfirmDialog`) are `rounded-window` on `bg-surface` with `shadow-2xl`. Their Cancel is an outlined button
- **Menus, popovers and toasts** are `rounded-control` with `shadow-lg`

## Rail Button Specs

Common specs for buttons placed on the left rail ([01 Layout](01-layout.md)).

The rail's buttons sit on the frame with no fill of their own, as the title bar's do ([meguri#585](https://github.com/zabuton-app/meguri/issues/585)).

### Content Items (Workspaces, etc.)

- Size: `size-11` (44px), `rounded-card`. The radius does not change on hover
- Unselected: `text-muted hover:bg-fg/10 hover:text-fg`
- Selected: `bg-primary/15 text-fg`, with a 3px bar at the rail's left edge (`before:absolute before:inset-y-3 before:-left-2.5 before:w-0.75 before:rounded-full before:bg-primary`), as VS Code's activity bar
- Content: an emoji, or initials (shared `initials()` that generates 2 alphanumeric characters / 1–2 Japanese characters), or an icon at `size-5`
- Deletable items show a × button at the top right on hover: `absolute -right-1 -top-1 hidden size-4 items-center justify-center rounded-full bg-error text-bg group-hover:flex`

### Add Button

`rounded-card border border-dashed border-border text-muted transition hover:border-primary hover:text-primary` + the `Plus` icon

### Fixed Actions at the Bottom

`rounded-card text-muted transition hover:bg-fg/10 hover:text-fg` + an icon at `size-5`. Always set `title` / `aria-label`
