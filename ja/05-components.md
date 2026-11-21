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
- Button のバリアントは `default / secondary / outline / ghost / destructive` × サイズ `default(h-9) / sm(h-8) / compact(h-7) / lg(h-10) / icon(h-9 w-9)` を維持する

## デザイントークン

すべての部品が値を取る段階。巡の`src/styles.css`で定義し、[巡のdocs/renderer.md](https://github.com/zabuton-app/meguri/blob/main/docs/renderer.md#design-tokens)で説明している（[meguri#584](https://github.com/zabuton-app/meguri/issues/584)）。段階から外れる値には理由が要る。

- **角の丸み（用途で名前を付ける）**: `rounded-panel`（16px、枠にはめ込んだ中身のパネル）、`rounded-window`（12px、モーダルとダイアログ）、`rounded-card`（8px、カード、パネル、一覧の行）、`rounded-control`（6px、ボタン、入力欄、メニュー、ポップアップ、バッジ、トースト）、`rounded-item`（4px、メニューの項目、小さなチップ、行やカードの中のサムネイル）、`rounded-full`（ピル）。内側の部品は外側より小さい丸みにする。`cn()`はこれらの名前を知るよう拡張してあり、`className`で渡した丸みが部品自身の丸みを置き換える
- **余白**: Tailwindの段階のうち0.5、1、1.5、2、3、4、6、8（2〜32px）
- **部品の高さ**: 24px（`h-6`、カードや行の中のアイコンボタン）、28px（`h-7`、`Button size="compact"`、ウィンドウの中の込み入った行やツールバー）、32px（`h-8`、`Button size="sm"`、`Input`、`Select`）
- **文字サイズ**: `text-2xs`（11px、バッジ、件数、補足の行。行の高さは持たない）、`text-xs`（UIの大半の文字）、`text-sm`（本文、メニューの項目、入力欄）、`text-base`（ウィンドウのタイトル）
- **枠線**: 1px。2pxはフォーカスと選択の縁取りだけ。区切りと入れ物は`border-border`、部品の縁は`border-border-strong`

### 状態

- **オン**: オンの部品（トグル、切り替えの選択中、押したフィルター、レールの選択中の項目）は`bg-primary/15`の薄い色を敷く。ベタ塗りにはしない。枠線は`border-border`のまま。お気に入りは赤の意味を残す（`bg-error/15 text-error`）
- **ホバー**: 枠線のないボタンや行は`bg-fg/10`、選択中のものは`bg-primary/25`
- **一覧の選択とフォーカス**: 選択は薄い色、フォーカスは2pxの`ring-primary`の縁取りで示し、両方を同時に出せるようにする。CSSテーマ向けに、選択中の項目には`data-selected`、フォーカス中の項目には`aria-current="true"`を付ける

### ウィンドウとダイアログ

- **ウィンドウの中に箱を作らない**: 全画面のウィンドウ（設定、ファイル詳細）のまとまりは、それぞれ箱に入れず、線（`border-b`／`border-t border-border`）で区切る。箱は、一覧、お知らせ、選択肢の集まりなど、中身を入れるものにだけ残す
- **小さいダイアログ**（共通の`Dialog`、`ConfirmDialog`）は、`bg-surface`の地に`rounded-window`と`shadow-2xl`。キャンセルは枠線付きのボタンにする
- **メニュー、ポップアップ、トースト**は`rounded-control`と`shadow-lg`

## レールボタン仕様

左レール（[01 レイアウト](01-layout.md)）に置くボタンの共通仕様。

レールのボタンは、タイトルバーのボタンと同じく、自分の地の色を持たずに枠の上に置く（[meguri#585](https://github.com/zabuton-app/meguri/issues/585)）。

### コンテンツ項目（ワークスペース等）

- サイズ: `size-11`（44px）、`rounded-card`。ホバーで丸みは変えない
- 非選択: `text-muted hover:bg-fg/10 hover:text-fg`
- 選択中: `bg-primary/15 text-fg`。レールの左端に3pxの縦の線を出す（`before:absolute before:inset-y-3 before:-left-2.5 before:w-0.75 before:rounded-full before:bg-primary`）。VS Codeの左端のバーと同じ見せ方
- 中身: 絵文字 or イニシャル（英数 2 文字 / 日本語 1〜2 文字を生成する `initials()` を共通利用）or アイコン `size-5`
- 削除可能な項目はホバーで右上に × ボタン: `absolute -right-1 -top-1 hidden size-4 items-center justify-center rounded-full bg-error text-bg group-hover:flex`

### 追加ボタン

`rounded-card border border-dashed border-border text-muted transition hover:border-primary hover:text-primary` + `Plus` アイコン

### 最下部の固定アクション

`rounded-card text-muted transition hover:bg-fg/10 hover:text-fg` + アイコン `size-5`。必ず `title` / `aria-label` を付ける
