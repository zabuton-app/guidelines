# 11 E2E Testing

## Principle

**E2E tests always run on Xvfb (a virtual X server).** This keeps windows off the real desktop and gives local runs and CI the same conditions. It applies to every test that launches the built app with Playwright's `_electron` (regardless of form: `@playwright/test` specs, UI check scripts under `tools/`, and so on).

## Pinning to X11 (Required)

`xvfb-run` only changes `DISPLAY`. When run from a Wayland session, Electron sees `ELECTRON_OZONE_PLATFORM_HINT=auto` and `XDG_SESSION_TYPE=wayland` and connects to Wayland. As a result, either a window appears on the real desktop or startup fails with `Failed to connect to Wayland display`. To prevent this, the launch helper does **both** of the following when running on Xvfb.

- **Launch argument**: Add `--ozone-platform=x11` to the `args` of `electron.launch()`. Environment variables alone sometimes have no effect; the command-line switch is the reliable one
- **Environment variables**: Remove `WAYLAND_DISPLAY` and `ELECTRON_OZONE_PLATFORM_HINT` from the child process `env`, and set `XDG_SESSION_TYPE=x11`

When the app is launched from more than one place (for example, a `spawn` to verify double-launch handling), route every one of them through the same helper. If even one is missed, a window appears on the real desktop only when running on Xvfb. This bug does not reproduce on a normal display, so it is hard to notice.

## Screen Settings

- **24-bit color depth is required**. The `xvfb-run` default is 640x480x8, which does not have enough color depth for transparent windows
- The default resolution is `1280x960`. Apps that need a larger screen, such as those that position themselves relative to the tray, may enlarge it (kizami uses `1920x1080x24`)
- Let `xvfb-run -a` (`--auto-servernum`) pick the server number automatically

```bash
xvfb-run -a -s "-screen 0 1280x960x24" -- npx playwright test
```

## npm Scripts

| Script | Description |
| ------ | ----------- |
| `test:e2e` | Builds, then runs E2E on the current display (for visual debugging) |
| `test:e2e:headless` | Builds, then runs E2E with the X11 pinning above on Xvfb. **Use this one normally** |

- The E2E job in CI also runs under the same conditions as `test:e2e:headless` (Xvfb, X11 pinning, 24-bit)
- Use `test:e2e:headless` when having an agent such as Claude Code run E2E as well

## Compliance Status of Existing Apps

As of 2026-09-30, the status is as follows. When fixing an app, do not align it on your own; confirm with the user first.

| App | Status |
| --- | ------ |
| 巡 (meguri) | Compliant (`test:e2e:headless` sets `MEGURI_FORCE_X11=1`, and the launch helper applies X11 pinning through both the argument and the environment variables. CI also uses `test:e2e:headless`; the screen is `1280x960x24`) |
| 刻 (kizami) | Compliant (X11 pinning is handled through both the argument and the environment variables; the screen is `1920x1080x24`) |
