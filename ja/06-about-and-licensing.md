# 06 About とライセンス

設定画面の最下部に About セクション（`src/routes/Settings/AboutSection.tsx`）を必ず置く。

## 必須項目

1. **アプリ名 + バージョン**: `about_info` IPC（`{ version, electron, chrome, node }` を返す `ipcMain.handle`）で取得し、`アプリ名 バージョン {version}` と `Electron / Chromium / Node` の各バージョンを表示する
2. **GitHub リンク**: `Button variant="outline"` + `ExternalLink` アイコン
3. **アプリ本体のライセンス表記**: 「<アプリ名> は MIT License の下で公開されています。」+ LICENSE ファイルへのリンク
4. **サードパーティライセンス一覧（THIRD_PARTY）**: 配布物にバンドルされるソフトウェアのみ列挙する（ビルドツールは含めない）。各エントリに名前・ライセンス種別バッジ・ライセンス / ソースへのリンクを付ける
5. **全依存パッケージへのリンク**: GitHub 上の package.json へのリンク

## 法的注意

- GPL 等のコピーレフトライセンスのバイナリをバンドルする場合（巡の FFmpeg 等）は、ライセンス文と対応ソースの入手先を必ず明示する
- 外部 URL はすべて `api.openUrl()`（メインプロセスの `shell.openExternal`、http/https のみ許可）経由で開く

## IPC 実装

```ts
ipcMain.handle("about_info", () => ({
  version: app.getVersion(),
  electron: process.versions.electron ?? "",
  chrome: process.versions.chrome ?? "",
  node: process.versions.node ?? "",
}));
```

preload の `INVOKE_CHANNELS` 許可リストに `"about_info"` を追加し、レンダラの `api.aboutInfo()` から呼ぶ。
