# 02 ナビゲーション

## ルーティング

- ルーターは `react-router` v8 の hash ルーティングを使う（`createHashRouter`。file:// ベースの Electron レンダラで安定するため）。v7 までの `react-router-dom` ではなく、単一パッケージの `react-router` を使う
- メイン画面は常時マウントし、詳細・設定などは**子ルートのモーダル**として上に重ねる（背後の画面の状態を保つ）

## ルーテッドモーダル

モーダルで開く画面（設定・詳細等）は URL ルートを持たせる。

- 開く: レールのボタン等から `navigate("<親パス>/settings")` する
- 閉じる: `navigate("..")`（または親パスへの `navigate`）で子ルートを外す。モーダルは Esc キーとバックドロップクリックの両方で閉じられること
- モーダル枠: `fixed inset-0 z-50` + オーバーレイ `bg-black/70 backdrop-blur-sm`

## レールの配置とルーター

レールの実装がルート状態に依存するかどうかで配置を選ぶ。

- **ルート状態に依存する場合**: レイアウトルートを作り、レールを `<Route element={<AppLayout />}>` の中に置く。レール内で `useParams` / `useNavigate` が使える

  ```tsx
  <Route element={<AppLayout />}>
    <Route path="/sheet/:id" element={<Sheet />}>
      <Route path="settings" element={<Settings />} />
    </Route>
  </Route>
  ```

- **ルート状態に依存しない場合（巡型）**: レールを `RouterProvider` の外に置き、`window.location.hash` 直接操作でナビゲートしてもよい

## 旧 URL の互換

画面の URL 構造を変えた場合、旧 URL からのリダイレクトルート（`<Route path="/settings" element={<Navigate to="/" replace />} />` 等）を残すこと。
