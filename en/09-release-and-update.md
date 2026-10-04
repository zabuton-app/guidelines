# 09 Release and Update Checks

## Versioning and Releases

- Versions follow **semver** (`X.Y.Z`). Release tags use the `vX.Y.Z` format
- Pre-releases are marked with the prerelease flag on GitHub

### Distribution Channels

Each app is expected to be distributed through the following three channels.

1. **GitHub Release** (the `zabuton-app/<app>` repository) — The primary channel. Binaries for every platform are distributed here
2. **AUR** (Arch User Repository) — For Linux. The package name is `<app>-bin`, registered as a binary package that takes the AppImage from the GitHub Release as its `source`. Declare `provides=('<app>')` / `conflicts=('<app>')`, and fetch the LICENSE from the raw URL of the corresponding release tag. The PKGBUILD is maintained in the `<app>-bin/` directory (e.g. [meguri-bin](https://github.com/zabuton-app/meguri-bin/blob/master/PKGBUILD))
3. **Microsoft Store** — For Windows. The Store handles updates for Store-distributed builds (see the destination branching described below)

## In-App Update Checks (Required for Electron Apps)

The reference implementation is meguri's [`electron/core/updater.ts`](https://github.com/zabuton-app/meguri/blob/main/electron/core/updater.ts). It is shared by copying; a new app replaces only the repository name, the Store ID, and the env variable name.

### Required Behavior

- **The check target is `/releases/latest` of GitHub Releases** (`https://api.github.com/repos/zabuton-app/<app>/releases/latest`). GitHub excludes drafts / pre-releases on the server side
- **Notification only. No automatic download or automatic installation**. The app only tells the user that an update exists and directs them to the release page (`html_url`, falling back to the releases list)
- **The automatic check at startup is throttled to 6 hours**. A manual check (the button on the settings screen) bypasses the throttle with `force`
- **The settings must include `autoCheck` (enable/disable automatic checks) and `ignoredVersion` (skip this version)**
- **The repository to check is hard-coded** (do not depend on `repository` in `package.json`). Only in development builds (`app.isPackaged === false`) can it be overridden with an env variable (`<APP>_UPDATE_REPO`); packaged builds ignore it
- **The destination (redirect URL) of a new version notification branches by distribution channel**
  - **Microsoft Store builds**: Navigate to the Store product page URL (`ms-windows-store://pdp/?ProductId=<ID>`), because the Store is responsible for applying updates. Determine whether the build is a Store build with `process.windowsStore`
  - **Every other platform** (including direct GitHub Release distribution and AUR): Navigate to the GitHub Release page of the corresponding release (`html_url`, falling back to the releases list)
  - When expanding to other platforms (stores) in the future, a destination suited to that channel may be added (it is not bound by this branching rule)
- Version comparison is a numeric dotted comparison with the `v` prefix stripped. Network failures and invalid responses are treated as "could not check (null)", distinct from "no update"
- Validate data received from GitHub against a schema before it crosses the IPC boundary
