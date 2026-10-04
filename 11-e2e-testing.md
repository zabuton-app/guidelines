# 11 E2Eテスト

## 原則

**E2Eテストは必ずXvfb（仮想Xサーバ）上で実行する。** 実デスクトップにウィンドウを出さないためと、ローカルとCIで同じ条件にそろえるため。対象は、Playwrightの`_electron`でビルド済みアプリを起動するテストすべて（`@playwright/test`のspec、`tools/`配下のUIチェックスクリプトなどの形式は問わない）。

## X11への固定（必須）

`xvfb-run`が変えるのは`DISPLAY`だけ。Waylandセッションから実行すると、Electronは`ELECTRON_OZONE_PLATFORM_HINT=auto`と`XDG_SESSION_TYPE=wayland`を見てWaylandに接続してしまう。その結果、実デスクトップにウィンドウが出るか、`Failed to connect to Wayland display`で起動に失敗する。これを防ぐため、Xvfbで実行するときは起動ヘルパーで次の2つを**両方**行う。

- **起動引数**: `electron.launch()`の`args`に`--ozone-platform=x11`を付ける。環境変数だけでは効かないことがあり、確実なのはコマンドラインスイッチのほう
- **環境変数**: 子プロセスの`env`から`WAYLAND_DISPLAY`と`ELECTRON_OZONE_PLATFORM_HINT`を消し、`XDG_SESSION_TYPE=x11`を設定する

アプリを起動する箇所が複数ある場合（二重起動の検証で`spawn`する等）は、全箇所で同じヘルパーを通す。1か所でも漏れると、Xvfb上でだけ実デスクトップに窓が出る。この不具合は通常のディスプレイでは再現しないので、気づきにくい。

## 画面設定

- 色深度は**24bit必須**。`xvfb-run`の既定値は640x480x8で、透過ウィンドウには色深度が足りない
- 解像度の既定は`1280x960`。トレイ基準で配置するなど、それより広い画面が要るアプリは広げてよい（刻は`1920x1080x24`）
- サーバ番号は`xvfb-run -a`（`--auto-servernum`）で自動採番させる

```bash
xvfb-run -a -s "-screen 0 1280x960x24" -- npx playwright test
```

## npmスクリプト

| スクリプト | 内容 |
| ---------- | ---- |
| `test:e2e` | ビルドしてから、現在のディスプレイでE2Eを実行する（目視でのデバッグ用） |
| `test:e2e:headless` | ビルドしてから、上記のX11固定とXvfbでE2Eを実行する。**通常はこちらを使う** |

- CIのE2Eジョブも`test:e2e:headless`と同じ条件（Xvfb・X11固定・24bit）で実行する
- Claude CodeなどのエージェントにE2Eを実行させるときも、`test:e2e:headless`を使う

## 既存アプリの準拠状況

2026-09-30時点で次のとおり。直すときは勝手に寄せず、ユーザーに確認してから対応する。

| アプリ | 状況 |
| ------ | ---- |
| 巡（meguri） | 準拠（`test:e2e:headless`が`MEGURI_FORCE_X11=1`を立て、起動ヘルパーでX11固定を引数・環境変数とも適用。CIも`test:e2e:headless`を使用、画面は`1280x960x24`） |
| 刻（kizami） | 準拠（X11固定は引数・環境変数とも対応済み、画面は`1920x1080x24`） |
