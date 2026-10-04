# 01 Layout

## App Shell

Every app uses a two-column structure: left rail + content.

```tsx
<div className="flex h-full">
  <AppRail />
  <div className="min-w-0 flex-1">{/* content */}</div>
</div>
```

- Always put `min-w-0 flex-1` on the content side (prevents accidental horizontal scrolling)
- Keep the OS-native title bar (do not set `frame: false`). Hide the menu with `autoHideMenuBar: true` + `Menu.setApplicationMenu(null)`
- **Exception**: An app with no navigation items, where the rail would hold only the logo and the settings button, may drop the left rail and consolidate everything into a single top bar. In that case, put the logo at the left end and the app-wide actions (settings button, etc.) at the right end
  - Even when there are multiple screens, this exception applies if there is no navigation back and forth between them and the transition is linear
  - In an app that already has a vertical tool palette inside the content area, adding a rail would put two vertical strips with different meanings side by side. That is exactly the confusion this chapter is meant to prevent, so the exception takes priority

## Left Rail (Slack Style)

Place a fixed-width vertical rail at the left edge of the screen. It is not collapsible.

```text
┌────┐
│ 🅰 │ ← App icon (top, fixed)
├────┤
│ ◻ │ ← Content-dependent items (scroll area)
│ ◻ │    meguri: workspaces / collections
│ ＋ │
├────┤ ← Divider h-px w-8 bg-border
│ ⚙ │ ← App-wide actions (bottom, fixed)
└────┘
```

- Container: `<nav className="flex h-full w-16 shrink-0 flex-col items-center gap-2 border-r border-border bg-bg py-3">`
- The width is fixed at `w-16` (64px)
- Put content-dependent items in a `ScrollArea` (`min-h-0 w-full flex-1`, `viewportClassName="px-1 pt-1"`) so that the rail itself does not grow even with many items
- Reordering items uses dnd-kit (`restrictToVerticalAxis` + `restrictToFirstScrollableAncestor`, `activationConstraint: { distance: 4 }`, `animateLayoutChanges: () => false`) and is allowed vertically only
- The fixed actions at the bottom must include a "Settings" button (`Settings` icon)

## Logo Placement

- Show the app logo at the top of the rail using `logo/app-256.png` with `size-[50px] shrink-0`. This is the only place the logo appears in the UI
- In an app without a left rail (the exception above), show `logo/app-256.png` at the left end of the top bar with `size-8 shrink-0`. The "one place only" principle still applies
- Put logo assets in each repository's `logo/` directory with shared names: `app-{32,64,128,256,512}.png`, `appicon-1024.png`, `tray-{16,32,64,256}.png`
- The tray icon is embedded as base64 (`TRAY_ICON_BASE64`) in `electron/main.ts`

## Tray Icon

**Every app must keep an icon in the system tray (the menu bar on macOS, the notification area on Windows).** This is required without exception, both for tray-centric apps (刻) and for window-centric apps (巡).

- Use `logo/tray-{16,32,64,256}.png` with a background on every platform (including macOS). Do not pass `--no-tray` when generating with the `icon-gen` skill (see [08 Logo](08-logo.md))
- Do not use a template image (`setTemplateImage(true)`) on macOS either. Turning a tray icon with a background into a template fills in the entire rounded background and makes the glyph invisible
- Put the app's display name (single kanji) in the tooltip
- Left click shows (or toggles) the window
- The context menu includes at least "Open" and "Quit". The labels are subject to i18n (see [07 i18n](07-i18n.md))
- Create the tray after `app.whenReady()` and `destroy()` it when the app quits

## App-Specific Panels

App-specific panels inside the content area (such as a `w-64` info panel on the right) are left to each app's discretion. However, they must not duplicate the role of the left rail (global navigation and the path to settings).
