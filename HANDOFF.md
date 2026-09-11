# 引き継ぎ書：青の時間（チル写真スクリーンセーバー）

## 目的

チルな音楽を流しながら、チルな写真をスクリーンセーバーのように眺めるための1枚もののWebページ。
音楽は本人の手持ちアルバムを使うので、**写真を充実させることと、写真の見せかたが鍵**。

## 方向性

- 吉村和敏の写真集の雰囲気。ブルーモーメント（夕暮れと夜のあいだ）、ヨーロッパの村、プリンス・エドワード島、雪の村と灯り、田園と空。
- 写真は「保存せず、その場で取ってきて眺める」。自分のサーバーに複製しない。
- XやInstagramの埋め込みは、投稿の枠やボタンが付くのでスクリーンセーバー用途には不向きと判断して不採用。
- 写真の出どころは以下の3つ：
  1. Wikimedia Commons（鍵不要、CORS可、`origin=*`）。当たり外れはあるが良い写真もある
  2. Unsplash API（無料のAccess Keyが必要、質が高い）
  3. 自分の写真フォルダ（端末内で完結、どこにも送らない）

## いま出来ているもの：`blue-moment.html`

依存なし、単一HTML。ブラウザで開くだけで動く。

### 見せかた
- 2枚のスライドを4秒のクロスフェードで切り替え
- ケン・バーンズ効果（ゆっくり寄る／引く、ランダムなパン）
- 「青の深さ」スライダー：青系グラデーションを `mix-blend-mode: multiply` で重ね、写真ごとの色味をそろえる（既定28%）
- 薄いフィルムグレイン（SVG feTurbulence）とビネット
- 左下に写真集風キャプション（地名／撮影者／出典）、右下に細い明朝の時計と日付
- 操作しないとカーソルとHUDが消える（3.5秒）
- `prefers-reduced-motion` 対応、モバイルではキャプションと時計を左に寄せる

### 機能
- テーマ5つをチェックボックスで組み合わせ（複数可）。テーマごとに英語の検索語リスト `THEMES` を持つ
- 切り替え間隔 8〜120秒
- 音楽：`<input type=file>` で手元の音源を選び、順番にループ再生（object URL、保存しない）
- 自分の写真：同様に `<input type=file multiple>`
- 設定はURLのハッシュ `#` にbase64 JSONで保存（localStorage不使用）。Unsplashのキーもここにだけあってコードやリポジトリには残らない
- キー操作：Space 停止／再開、→ 次、F 全画面、C キャプション、S 設定、Esc 設定を閉じる

### 取得ロジック
- `refill()` が在庫が6枚を切ったら検索語を順に回して補充。既出は `seen` で除外、600枚超えたらリセット
- Commonsは `width >= 1600` かつ横長（比率1.15以上）だけ採用、`iiurlwidth=2400` のサムネURLを使う
- Unsplashは `orientation=landscape`, `content_filter=high`, ページ1〜3をランダム
- 出どころやテーマを変えると `generation` を進めて古い取得結果を捨てる
- 読み込みに失敗した画像は飛ばす

### 未検証
- 作成環境がCommonsのAPIにレート制限されていたため、実機での動作確認はまだ。ブラウザから普通に使う分には問題ないはずだが、最初に確認してほしい
- Commonsの検索語の当たり具合。`THEMES` の語は調整前提

## GitHub Pages 化（済み）

リポジトリ: https://github.com/hogasahara/screen
公開 URL: https://hogasahara.github.io/screen/

- `blue-moment.html` は `index.html` としてリポジトリ直下に置いた
- `manifest.json`（`display: fullscreen`、テーマカラー `#0a1224`）と `icon.svg` / `icon-192.png` / `icon-512.png` を足し、`index.html` の `<head>` にマニフェスト・theme-color・apple-touch-icon を追加した。「ホーム画面に追加」で全画面起動できる
- 公開は GitHub Actions（`.github/workflows/pages.yml`）。`main` への push で `index.html`・`manifest.json`・アイコンだけを `_site/` に集め、`actions/upload-pages-artifact` → `actions/deploy-pages` で配置する
- README に使いかた（開く→F→設定でテーマ、音楽と写真は手元から）を書いた
- アイコンの PNG は `icon.svg` をヘッドレス Chromium で撮ったもの。直したいときは SVG を編集して撮り直す

### 初回だけ手でやること
- リポジトリの Settings → Pages → Build and deployment → Source を **GitHub Actions** にする。これをしないとワークフローの deploy ステップが失敗する
- 作業ブランチ `claude/loving-johnson-vqalf9` を `main` にマージすると最初の配置が走る

### まだ残っているもの
- 実機で動作確認し、Commons の検索語（`THEMES`）を調整する
- 「ホーム画面に追加」の実機確認（iOS は `apple-touch-icon` の PNG、Android は manifest のアイコンを使う）

## その先の候補

- Unsplashのコレクション指定（検索より雰囲気がそろう）
- お気に入り登録して、そこだけから出すモード
- 時間帯でテーマや青の深さを自動で変える（夕方は青を深く、など）
- 自分の写真とWebの写真を混ぜる比率
- キャプション位置や書体の調整

## 注意

- UnsplashのAccess Keyをコードやコミットに入れない（今の設計ならURLにしか残らない）
- Commonsの写真はCC系ライセンスなので、キャプションの撮影者・ライセンス表示は消さない
- Unsplashは表示時に撮影者名を出す規約。キャプションを消せる設定はあるが、既定では出しておく
