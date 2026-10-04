# 05 コンポーネント

## 共有 UI コンポーネント（shadcn/ui パターン）

- `src/components/ui/` に shadcn/ui パターン（Radix + CVA）の自前実装を置く: `button` / `dialog` / `select` / `dropdown-menu` / `scroll-area` / `input` / `skeleton` 等
- クラス結合は `cn()`（`clsx` + `tailwind-merge`、`src/lib/utils.ts`）を使う
- アイコンは lucide-react、トーストは sonner、並び替えは dnd-kit で統一する
- **確認ダイアログはアプリ内モーダルの `ConfirmDialog`（`useConfirm()` が `Promise<boolean>` を返す）を使う**。`window.confirm` / `window.alert` 等のネイティブダイアログは禁止（メインウィンドウ以外のウィンドウを開く導線を作らない）。`ConfirmProvider` をアプリルートに置く

### コピー運用ルール

- 共有コンポーネントはアプリ間でコピー同期する。**コピー後に独自の差分を作らない**（整形差分も避ける）
- 改善はまず巡側に入れ、他アプリへ展開する
- Button のバリアントは `default / secondary / outline / ghost / destructive` × サイズ `default(h-9) / sm(h-8) / lg(h-10) / icon(h-9 w-9)` を維持する

## レールボタン仕様

左レール（[01 レイアウト](01-layout.md)）に置くボタンの共通仕様。

### コンテンツ項目（ワークスペース等）

- サイズ: `size-11`（44px）
- 非選択: `rounded-2xl bg-surface text-fg hover:rounded-xl hover:bg-overlay`（Slack 風の角丸アニメーション）
- 選択中: `rounded-xl bg-primary text-primary-foreground ring-2 ring-fg/40`
- 中身: 絵文字 or イニシャル（英数 2 文字 / 日本語 1〜2 文字を生成する `initials()` を共通利用）or アイコン `size-5`
- 削除可能な項目はホバーで右上に × ボタン: `absolute -right-1 -top-1 hidden size-4 items-center justify-center rounded-full bg-error text-bg group-hover:flex`

### 追加ボタン

`rounded-2xl border border-dashed border-border text-muted transition hover:rounded-xl hover:border-primary hover:text-primary` + `Plus` アイコン

### 最下部の固定アクション

`rounded-2xl text-muted transition hover:rounded-xl hover:bg-overlay hover:text-fg` + アイコン `size-5`。必ず `title` / `aria-label` を付ける
