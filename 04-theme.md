# 04 テーマ

## カラーシステムは Base16 を正とする

全テーマは [base16](https://github.com/chriskempson/base16) の 16 色スキームとして定義し、セマンティックトークンを経由して UI に適用する。コンポーネントは生の色値を持たず、必ずセマンティックトークンを参照すること。

### 3 段ブリッジ

```text
base16 スキーム → --c-*（実行時 CSS 変数） → --color-*（Tailwind） → コンポーネント
```

1. `src/themes/base16.ts` の `SEMANTIC_MAP` が base16 → セマンティック名を対応付ける:
   `bg`(base00), `surface`(01), `overlay`/`border`(02), `muted`(03), `secondary-fg`(04), `fg`(05), `bright-fg`(06), `highlight`(07), `error`(08), `warn`(09), `accent2`(0A), `success`(0B), `info`(0C), `primary`(0D), `secondary-accent`(0E), `special`(0F)
2. `schemeToCssVars()` が `[data-theme="<id>"]` ブロックとして `--c-*` を生成し、`ThemeProvider` が単一の `<style>` に注入する
3. `src/styles.css` の `@theme inline` で `--color-*: var(--c-*)` にブリッジし、`bg-bg` / `text-fg` / `border-border` 等のユーティリティとして使う。shadcn 互換エイリアス（`--color-background`、`--color-primary-foreground` 等）も同ブロックで定義する

## 対応必須テーマファミリー

主要テーマとして以下の 12 ファミリーを全アプリで共通サポートする。各ファミリーは light / dark ペアを持つこと（計 24 スキーム）。

| ファミリー | 備考 |
| --- | --- |
| Gruvbox | デフォルトテーマ（`gruvbox-dark`） |
| Solarized | |
| Nord | |
| Catppuccin | |
| Tokyo Night | |
| Ayu | |
| Material | |
| Monokai | |
| GitHub | |
| One | |
| Rosé Pine | |
| Tomorrow | |

## テーマ定義の共通化

- `src/themes/base16.ts`、`src/themes/schemes.ts`、`src/themes/ThemeProvider.tsx` はシリーズ共通コードとし、アプリ間でコピー同期する（差分を作らない。localStorage キーの `<app>.theme` プレフィックスと `<style>` 要素 id のみアプリ名に置換可）
- 新テーマの追加はまず巡の `schemes.ts` に追加し、全アプリへ展開する

## 切替と永続化

- ダークモードは `prefers-color-scheme` ではなく `<html data-theme="<id>">` 属性の切替で実現する。各スキームは `appearance: "light" | "dark"` を持ち、`setMode` でファミリー内ペアを切り替える
- 永続化は `localStorage`（キー: `<app>.theme`、例 `meguri.theme`）
- FOUC 対策として `public/theme-boot.js` を `index.html` の `<head>` で読み込み、初回描画前に `data-theme` を適用する

## テーマプレビュー

設定画面のテーマカードには `deriveTokens()` を通した**セマンティックトークン 6 色**（`bg` / `border` / `muted` / `primary` / `accent2` / `error`）のスウォッチを表示する。

生の palette スロットではなく派生後の値を使うのは、スウォッチが「実際に UI で使われる色」と一致する必要があるため。上流の base16 スキームには非単調なランプや、スロットをアクセントに転用しているものがあり、生の値を並べると一部テーマで見えなくなる chrome 色（hairline・二次テキスト）を切り替える前に判断できない。

実装は巡の [`src/routes/Settings/index.tsx`](../meguri/src/routes/Settings/index.tsx) の `PREVIEW_TOKENS` を正とする。
