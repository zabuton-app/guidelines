# Zabuton Series Guidelines

A set of rules for keeping brand expression, screen UI, and release operations consistent across every app under the 座布団 (zabuton) brand. Follow these guidelines whenever you create a new app or change an existing one.

The English version is the source of truth. A Japanese translation is available in [ja/](ja/README.md).

## Scope

- [巡 (meguri)](https://github.com/zabuton-app/meguri) — Local media browser. The reference implementation of these guidelines
- [刻 (kizami)](https://github.com/zabuton-app/kizami) — Pomodoro timer that lives in the tray
- Every app added to the zabuton series in the future

## Chapters

- [00 Brand and Naming](00-brand.md) — Series name, app naming rules, and display name conventions
- [01 Layout](01-layout.md) — App shell (left rail + content), logo placement, and tray icon
- [02 Navigation](02-navigation.md) — Conventions for routing and routed modals
- [03 Settings Screen](03-settings.md) — Settings modal dimensions, section structure, and row pattern
- [04 Theme](04-theme.md) — Base16 color system and required themes
- [05 Components](05-components.md) — Shared UI components and rail button specs
- [06 About and Licensing](06-about-and-licensing.md) — Required items in the About section
- [07 i18n](07-i18n.md) — Translation key naming conventions and the Provider API shape
- [08 Logo](08-logo.md) — Composition, colors, and export sizes of the single-kanji logo
- [09 Release and Update Checks](09-release-and-update.md) — Versioning, distribution, and in-app update notifications
- [10 Writing Language](10-language.md) — Repository artifacts are in English, plus the list of exceptions written in Japanese
- [11 E2E Testing](11-e2e-testing.md) — E2E tests run on Xvfb, pinned to X11
- [12 Mascot](12-mascot.md) — Role of the official character, where its assets live, and which variant to use per background

## Core Principles

- **Unified tech stack**: Electron + React 19 + TypeScript + Tailwind CSS v4 + shadcn/ui (Radix) + lucide-react + TanStack Query
  - TanStack Query is only for apps that have server state or asynchronous reads requiring caching and invalidation. Apps without them do not need to adopt it
- **Repositories are written in English**: Code, documentation, commits, and issues/PRs are written in English. See [10 Writing Language](10-language.md) for the exceptions
- **meguri is the reference**: When in doubt, treat the meguri implementation as correct. Shared code is maintained by copying (not packaged), and no app-specific differences are introduced after copying
- **Align the file structure**: Files with the same role go in the same place under the same name, such as `src/components/<App>Rail.tsx`, `src/routes/Settings/{index,SettingsModal,AboutSection}.tsx`, and `src/themes/{base16,schemes,ThemeProvider}.ts(x)`
