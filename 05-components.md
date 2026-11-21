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
- Keep the Button variants `default / secondary / outline / ghost / destructive` × sizes `default(h-9) / sm(h-8) / lg(h-10) / icon(h-9 w-9)`

## Rail Button Specs

Common specs for buttons placed on the left rail ([01 Layout](01-layout.md)).

### Content Items (Workspaces, etc.)

- Size: `size-11` (44px)
- Unselected: `rounded-2xl bg-surface text-fg hover:rounded-xl hover:bg-overlay` (Slack-style corner radius animation)
- Selected: `rounded-xl bg-primary text-primary-foreground ring-2 ring-fg/40`
- Content: an emoji, or initials (shared `initials()` that generates 2 alphanumeric characters / 1–2 Japanese characters), or an icon at `size-5`
- Deletable items show a × button at the top right on hover: `absolute -right-1 -top-1 hidden size-4 items-center justify-center rounded-full bg-error text-bg group-hover:flex`

### Add Button

`rounded-2xl border border-dashed border-border text-muted transition hover:rounded-xl hover:border-primary hover:text-primary` + the `Plus` icon

### Fixed Actions at the Bottom

`rounded-2xl text-muted transition hover:rounded-xl hover:bg-overlay hover:text-fg` + an icon at `size-5`. Always set `title` / `aria-label`
