# 06 About and Licensing

Always place an About section (`src/routes/Settings/AboutSection.tsx`) at the bottom of the settings screen.

## Required Items

1. **App name + version**: Fetch via the `about_info` IPC (an `ipcMain.handle` that returns `{ version, electron, chrome, node }`) and show `<App name> version {version}` along with the versions of `Electron / Chromium / Node`
2. **GitHub link**: `Button variant="outline"` + the `ExternalLink` icon
3. **License notice for the app itself**: "\<App name\> is released under the MIT License." + a link to the LICENSE file
4. **Third-party license list (THIRD_PARTY)**: List only software bundled into the distributed artifacts (do not include build tools). Each entry has a name, a license type badge, and links to the license / source
5. **Link to all dependencies**: A link to package.json on GitHub

## Legal Notes

- When bundling binaries under a copyleft license such as the GPL (e.g. FFmpeg in meguri), always state the license text and where to obtain the corresponding source
- Open every external URL through `api.openUrl()` (`shell.openExternal` in the main process, http/https only)

## IPC Implementation

```ts
ipcMain.handle("about_info", () => ({
  version: app.getVersion(),
  electron: process.versions.electron ?? "",
  chrome: process.versions.chrome ?? "",
  node: process.versions.node ?? "",
}));
```

Add `"about_info"` to the `INVOKE_CHANNELS` allowlist in preload and call it from the renderer via `api.aboutInfo()`.
