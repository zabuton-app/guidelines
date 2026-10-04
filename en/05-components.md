# 05 Components

## Shared UI Components (shadcn/ui Pattern)

- Put in-house implementations of the shadcn/ui pattern (Radix + CVA) in `src/components/ui/`: `button` / `dialog` / `select` / `dropdown-menu` / `scroll-area` / `input` / `skeleton`, etc.
- Join classes with `cn()` (`clsx` + `tailwind-merge`, `src/lib/utils.ts`)
- Standardize on lucide-react for icons, sonner for toasts, and dnd-kit for reordering
- **For confirmation dialogs, use the in-app modal `ConfirmDialog` (`useConfirm()` returns a `Promise<boolean>`)**. Native dialogs such as `window.confirm` / `window.alert` are prohibited (do not create a path that opens any window other than the main window). Put `ConfirmProvider` at the app root

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
