# 05 コンポーネント

## 共有 UI コンポーネント（shadcn/ui パターン）

- `src/components/ui/` に shadcn/ui パターン（Radix + CVA）の自前実装を置く: `button` / `dialog` / `select` / `dropdown-menu` / `scroll-area` / `input` / `skeleton` 等
- クラス結合は `cn()`（`clsx` + `tailwind-merge`、`src/lib/utils.ts`）を使う
- アイコンは lucide-react、トーストは sonner、並び替えは dnd-kit で統一する
- **確認ダイアログはアプリ内モーダルの `ConfirmDialog`（`useConfirm()` が `Promise<boolean>` を返す）を使う**。`window.confirm` / `window.alert` 等のネイティブダイアログは禁止（メインウィンドウ以外のウィンドウを開く導線を作らない）。`ConfirmProvider` をアプリルートに置く

### スクロールバー

- **画面に見えるスクロールバーはすべて共通の`ScrollArea`（`src/components/ui/scroll-area.tsx`）にする**。プラットフォーム標準のスクロールバーを画面に残さない。`overflow-y-auto` / `overflow-auto`を付けた要素はOSのスクロールバーを描画し、テーマに追従せずプラットフォームごとに見た目も変わる
- 大きさはビューポートではなくルートに指定する。flexの縦並び（モーダル本文、タブパネル）の中では`min-h-0 flex-1`。コンテンツの余白は`viewportClassName`に書く
- `max-h-*`で高さを制限する領域は、その指定を`viewportClassName`に書く。ルートに付けるとスクロールせずに切り取られる
- コンテンツが短いときもビューポートいっぱいに広げたい場合（下端まで伸ばすペインの罫線、`sticky bottom-0`で固定するフッター）は、`fillViewport`を渡し、コンテンツ側に`flex-1`を指定する。Radixはコンテンツを`display: table`の箱で包むため、コンテンツ自体に`min-h-full`を付けても効かない。`fillViewport`がその箱のスタイルを直す。この箱の存在は`scroll-area.tsx`の中だけの知識とし、画面側から`[&>div]:`セレクタで触らない
- バーを見せる必要のない小さなはみ出し（狭いウィンドウで収まらないタブ列やセグメントコントロール）は`overflow-x-auto`のままにし、`src/styles.css`の`no-scrollbar`クラスでバーを隠す
- `@tanstack/react-virtual`で仮想化するリストは、`ScrollArea`に`viewportRef`を渡して`getScrollElement`に使う

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
