# 03 設定画面

設定画面はルーテッドモーダル（[02 ナビゲーション](02-navigation.md)）として実装する。実装は `src/routes/Settings/` に置き、`index.tsx`（画面の組み立て）＋ `SettingsModal.tsx`（枠）＋ セクション別ファイル（`AboutSection.tsx` 等）に分ける。セクションが状態やイベント購読を持つ場合は必ず別ファイルにし、`index.tsx` には行パターンのマークアップだけを残す。

## モーダル寸法

- オーバーレイ: `fixed inset-0 z-50 flex items-center justify-center bg-black/70 p-4 backdrop-blur-sm`
- パネル: `relative flex h-[85vh] w-full max-w-xl flex-col overflow-hidden rounded-xl border border-border bg-bg shadow-2xl`
- ヘッダー: タイトル + 右端に閉じるボタン（`X` アイコン、`title` に「閉じる (Esc)」）。`border-b border-border bg-bg px-4 py-2.5` で固定
- 本文: `ScrollArea`（`min-h-0 flex-1`、`viewportClassName="p-4 pb-6"`）内に `mx-auto flex max-w-2xl flex-col gap-8`

## セクション順序

既定は単一スクロールのセクション積みとする。順序は以下で固定。

1. Language（言語）
2. アプリ固有の設定（巡: Scenes / Keybinding 等）
3. Appearance（ライト / ダーク）
4. Theme（テーマファミリー選択）
5. Update（自動更新があるアプリのみ）
6. Support（寄付リンク、任意）
7. About（必須。[06 About とライセンス](06-about-and-licensing.md)を参照）

### 例外: タブ構成

設定項目が増えて単一スクロールでは目的の項目に辿り着けなくなったアプリは、カテゴリ別のタブに分割してよい。巡が該当し、`src/routes/Settings/SettingsTabs.tsx` として実装されている（2026-08-29 にタブ化、2026-09-23 に本項を追記）。

タブに分ける場合も次は守る。

- モーダル寸法・行パターン・コントロールの選択基準は上記のまま変えない
- タブ内のセクション順序は、上の既定順を崩さない範囲でカテゴリへ割り振る。About は最後のタブの末尾に置く
- タブの選択状態は永続化せず、開くたびに先頭のタブから始める
- 新しいアプリはまず単一スクロールで始める。最初からタブに分けない

## 行パターン

各セクションは「左にラベル + 説明、右にコントロール」の行パターンで統一する。

```tsx
<section className="flex items-center justify-between gap-3 rounded-md border border-border bg-surface px-4 py-3">
  <div className="flex flex-col">
    <span className="text-sm font-semibold text-bright-fg">{/* ラベル */}</span>
    <span className="text-xs text-muted">{/* 説明 */}</span>
  </div>
  {/* コントロール */}
</section>
```

## コントロールの選択基準

- **選択肢がリスト**（言語、プリセット等）: Radix ベースの `Select`（`@/components/ui/select`）
- **二者択一**（ライト / ダーク）: セグメントトグル。選択中は `bg-primary text-primary-foreground`、非選択は `bg-bg text-muted hover:text-fg`。アイコン（`Sun` / `Moon`）付き
- **テーマファミリー**: カードグリッド（`grid grid-cols-1 gap-2 sm:grid-cols-2`）。各カードに 6 色スウォッチ（`deriveTokens()` を通したセマンティックトークン。[04 テーマ](04-theme.md)のテーマプレビュー参照）+ ファミリー名、選択中は `border-primary ring-1 ring-primary` + `Check` アイコン
- **自由記述の複数行**（巡: AI のタグ語彙）: 全幅の `textarea`。`font-mono text-xs`・`resize-y`・`spellCheck={false}`・`aria-label` にラベルと同じ文言
- **連続値**（巡: AI のしきい値）: 全幅の `input type="range"`。現在値はラベル行の右端に `font-mono` で表示する。ネイティブの range は `role="slider"` と矢印キー操作を最初から持つので、独自実装しない
- **同種の項目が並ぶ一覧**（巡: AI のモデル一覧）: 1 列のリスト。各行は `rounded-md border border-border bg-bg px-3 py-2` で、左に名称 + 副次情報、右にアクション。選択中の行は `Check` アイコン + 選択中ラベルを添える

### 行パターンの例外

コントロールが全幅を要する場合（`textarea` / `range` / 一覧）は、ラベル + 説明を上、コントロールを下にした縦積みにしてよい。その場合もラベル + 説明のマークアップは行パターンと同一にする。

### 保存のタイミング

設定は即時反映を原則とする。反映に重い再計算を伴うもの（巡: AI のタグ語彙・しきい値は、変更をタグへ反映するのにライブラリ全件の再タグ付けが必要）に限り、明示的な保存ボタンを置き、未反映であることを警告として表示する。
