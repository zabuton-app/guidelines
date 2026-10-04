# 09 リリースと更新確認

## バージョニングとリリース

- バージョンは **semver**（`X.Y.Z`）。リリースタグは `vX.Y.Z` 形式
- プレリリースは GitHub 上で prerelease フラグを付ける

### 配布チャネル

各アプリは以下の 3 チャネルでの配布を前提とする。

1. **GitHub Release**（`zabuton-app/<app>` リポジトリ） — 基本チャネル。全プラットフォームのバイナリをここで配布する
2. **AUR**（Arch User Repository） — Linux 向け。パッケージ名は `<app>-bin` とし、GitHub Release の AppImage を `source` に取るバイナリパッケージとして登録する。`provides=('<app>')` / `conflicts=('<app>')` を宣言し、LICENSE は該当リリースタグの raw URL から取得する。PKGBUILD は `<app>-bin/` ディレクトリで管理する（例: [meguri-bin](https://github.com/zabuton-app/meguri-bin/blob/master/PKGBUILD)）
3. **Microsoft Store** — Windows 向け。Store 配布ビルドの更新は Store が担う（後述の誘導先分岐を参照）

## アプリ内の更新確認（Electron アプリ必須）

リファレンス実装は巡の [`electron/core/updater.ts`](https://github.com/zabuton-app/meguri/blob/main/electron/core/updater.ts)。コピー運用とし、新アプリではリポジトリ名・Store ID・env 変数名だけを差し替える。

### 必須の挙動

- **確認先は GitHub Releases の `/releases/latest`**（`https://api.github.com/repos/zabuton-app/<app>/releases/latest`）。GitHub が draft / prerelease をサーバー側で除外してくれる
- **通知のみ。自動ダウンロード・自動インストールはしない**。「更新がある」ことを知らせ、リリースページ（`html_url`、フォールバックは releases 一覧）へ誘導するだけ
- **起動時の自動チェックは 6 時間スロットル**。手動チェック（設定画面のボタン）は `force` でスロットルをバイパスする
- **設定として `autoCheck`（自動確認の有効/無効）と `ignoredVersion`（このバージョンをスキップ）を必ず持つ**
- **確認先リポジトリはハードコード**する（`package.json` の `repository` に依存しない）。開発ビルド（`app.isPackaged === false`）に限り env 変数（`<APP>_UPDATE_REPO`）で上書き可能とし、パッケージ済みビルドでは無視する
- **新バージョン通知からの遷移先（誘導 URL）は配布チャネルで分岐する**
  - **Microsoft Store 配布ビルド**: Store の製品ページ URL（`ms-windows-store://pdp/?ProductId=<ID>`）へ遷移させる。更新の適用は Store が担うため。Store ビルドかどうかは `process.windowsStore` で判定する
  - **それ以外のプラットフォーム**（GitHub Release 直配布・AUR 含む）: 該当リリースの GitHub Release ページ（`html_url`、フォールバックは releases 一覧）へ遷移させる
  - 将来他のプラットフォーム（ストア）へ展開する場合は、そのチャネルに適した誘導先を追加してよい（この分岐ルールに縛られない）
- バージョン比較は `v` プレフィックスを剥がした数値ドット比較。ネットワーク失敗・不正レスポンスは「確認できなかった（null）」として扱い、「更新なし」と区別する
- GitHub から受け取ったデータは IPC 境界を越える前にスキーマ検証する
