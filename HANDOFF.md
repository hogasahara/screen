# 引き継ぎ書：青の時間（チル写真スクリーンセーバー）

## 目的

チルな音楽を流しながら、チルな写真をスクリーンセーバーのように眺めるための1枚もののWebページ。
音楽は本人の手持ちアルバムを使うので、**写真を充実させることと、写真の見せかたが鍵**。

## 方向性

- 吉村和敏の写真集の雰囲気。ブルーモーメント（夕暮れと夜のあいだ）、ヨーロッパの村、プリンス・エドワード島、雪の村と灯り、田園と空。
- 写真は「保存せず、その場で取ってきて眺める」。自分のサーバーに複製しない。
- XやInstagramの埋め込みは、投稿の枠やボタンが付くのでスクリーンセーバー用途には不向きと判断して不採用。
- 写真の出どころは以下の4つ：
  1. Wikimedia Commons（鍵不要、CORS可、`origin=*`）。当たり外れはあるが良い写真もある
  2. Unsplash API（無料のAccess Keyが必要、質が高い）
  3. Flickr API（無料のAPI Keyが必要、CORS可）。CC系ライセンスだけ、`sort=interestingness-desc`。街のスナップが厚いので「何でもない日本」向き
  4. 自分の写真フォルダ（端末内で完結、どこにも送らない）

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
- テーマ7つをチェックボックスで組み合わせ（複数可）。テーマごとに英語の検索語リスト `THEMES` を持つ。ブルーモーメント／ヨーロッパの村／プリンス・エドワード島／雪と灯り／田園と空／アンコモン・プレイス（アメリカ）／何でもない日本の街
- `plain: true` のテーマ（アメリカ、日本の街）は認定カテゴリを探さず通常検索だけ。語ごとに段階リストを持つ（`activeQueries()` が `{q, tiers}` を返す）
- 切り替え間隔 8〜120秒
- 音楽：`<input type=file>` で手元の音源を選び、順番にループ再生（object URL、保存しない）
- 自分の写真：同様に `<input type=file multiple>`
- 設定はURLのハッシュ `#` にbase64 JSONで保存（localStorage不使用）。Unsplashのキーもここにだけあってコードやリポジトリには残らない
- キー操作：Space 停止／再開、→ 次、F 全画面、C キャプション、S 設定、Esc 設定を閉じる

### 取得ロジック
- 取りに行く先は `feeds`（検索語 × 段階）。Commons の段階は 秀逸な画像（`incategory:"Featured pictures on Wikimedia Commons"`）→ 品質画像（`incategory:"Quality images"`）→ 通常検索（関連度順）。上の段階に残りがある限りそこから取る
- 各 feed は取ってきた写真を自分の `buffer` に持つ。`gsroffset` で続きページを読み、段階ごとに上限（秀逸200・品質400・通常80件。関連度順は後ろほど外れるので通常は浅く）。テーマに `deep` があればその語はそこまで読む（Margolies は400）。Unsplash は段階なし、ページ1〜5
- 同じ写真が jpg と tif で二重に入っていることがあるので、タイトル（`dupKey`）でも重複を除く
- `refill()` は在庫が6枚を切ったら、いちばん上の段階で残りのある語をぜんぶ対象に、バッファが空の語を並列で取りに行き（最多6語）、各語のバッファから均等に1枚ずつ取って12枚ほどをシャッフルして在庫に入れる。以前は「1語で8枚集まったら終わり」だったので、1つの語（スナック）が20枚以上続いた。モックで測ると同じ語の連続は最大2枚
- 既出は `seen` で除外（秀逸は品質にも含まれるので重複が多い）
- 表示した写真は `history`（id → photo）に残す。全 feed を使い切って在庫が空になったら `recycle()` が履歴をシャッフルして在庫に戻し、繰り返す。直近24枚（履歴の半分まで）は後ろに回して、同じ写真が続かないようにする
- 自分のフォルダも、全部見せてから混ぜ直す巡回にした
- Commonsは `width >= 1600` かつ横長（比率1.15以上）だけ採用、`iiurlwidth=2400` のサムネURLを使う。キャプションのクレジット先頭に「秀逸な画像」「品質画像」を付ける
- 設定画面の状態表示は「◯枚を見つけました（秀逸 a・品質 b・ほか c）」、一巡後は「◯枚を一巡しました。繰り返しています。」
- 出どころやテーマを変えると `generation` を進めて古い取得結果を捨て、履歴も作り直す
- 読み込みに失敗した画像は飛ばす（履歴にも入れない）

### テーマの検索語について
- Commons の検索は語をすべて含むものだけ返す（AND）。語が多いほど外れる。「lake village」は品質画像だけで144件、「hilltop village」は1件
- 認定カテゴリ（秀逸・品質）は風景系には多いが、絵葉書的な構図に偏る。「何でもない場所」（スティーブン・ショア『アンコモン・プレイス』の方向）には向かないので、そこは `plain` で飛ばす
- 日本のレトロな街を地名（銀山温泉、奈良井宿、倉敷、尾道…）で引く案は、テンプレ観光地になるので不採用になった
- アメリカの「何でもない場所」は Commons 内の議会図書館アーカイブ、John Margolies のロードサイド写真（約9800枚、パブリックドメイン）に絞った。1970〜80年代のフィルムの色で正面から撮ったモーテル・ガソリンスタンド・レストラン・劇場・店・ドライブイン・キャビン。彼自身が作家なので200件目以降でも作品のまま（コンタクトシートで確認）。語は `"John Margolies" motel` のようにフレーズで作者名を入れる。`sign` は看板の寄りばかり、`miniature golf` も寄りが多いので外した
- Carol M. Highsmith は建築記録や内装が混ざり、認定写真は風景7枚だけだったので外した。`main street texas` `diner exterior` も現代の記録写真や古写真が混ざるので外した
- ファイル名の `LCCN…` と `by John Margolies` は `prettyName()` で落とす。`incategory:` や `deepcat:` でアーカイブのカテゴリを指定しても構造の都合で当たらないので、撮影者名を語に含めるほうが確実
- 環境保護庁の DOCUMERICA（1970年代）も同時代だが、NARA のスキャンは退色が強く同じ写真の jpg/tif 重複も多いので入れていない
- 実機で「精度がいまいち」という評価。原因は2つ。上位7枚は良くてもページ送りで深く読むと崩れる語があること（通常検索を80件までにして対処）と、Commons のアマチュア記録写真には「何でもない場所」ではあっても作品としての構図と光の意図がないこと。本物さと雑さは別物で、日本側はここが限界
- 日本の「何でもない街」は Commons に狙って撮る人が少ない。当たるのは、シリーズで上げている投稿者の語：`danchi`（団地の街路）、`japan residential street`（青山の住宅街シリーズ）、`"street view" fukuoka`（福岡の道路）、`japan port town street`（常神の港町）、`japan level crossing street`（踏切）、`incategory:"Snack bars in Japan"`（スナックの看板と路地）。`snack bar japan` は菓子が、`japan shopping street evening` は香港が混ざるので使わない
- 語の当たりを目で確かめる方法：API で `iiurlwidth=480` のサムネを取って HTML に並べ、ヘッドレス Chromium でスクリーンショットにする（コンタクトシート）。使える枚数だけでは雰囲気は分からない
- Flickr を出どころとして実装した（`fetchFlickr`）。`flickr.photos.search` に `text`、`license=1,2,3,4,5,6,7,9,10`、`sort=interestingness-desc`、`extras=url_l,url_h,url_k,owner_name,license`。2048（k）→1600（h）→1024（l）の順であるいちばん大きいサイズを使い、1024 未満と縦位置は捨てる。ページは8まで
- 語の先頭に `group:<グループID>` と書くとそのグループプールの中だけを探す（`group_id`）。「New Topographics」「Uncommon Places」系のグループ ID は鍵を得てから `flickr.groups.search` で調べて `THEMES[*].flickr` に足す
- テーマごとに `flickr` の語を持つ。無いテーマは `unsplash` の語で代用（`activeQueries()`）
- 鍵が無い・断られたときは `halted` を立てて取りに行くのを止める。鍵を入れ直すか出どころを変えると `restartSource()` で解ける。以前は5秒ごとに全語を叩き続けていた
- 作成環境では Flickr・Unsplash の鍵が無いので、実データでの当たり具合は未確認。モック（API 応答を偽装）で流れだけ確認した
- 使用回数順（`gsrsort=incoming_links_desc`）は記事に載せやすい記録写真が上に来るので審美性とは無関係。使っていない
- 語の当たりを数えるには `gsrinfo=totalhits` を付けて API を叩く。連続で叩くとレート制限で空応答が返るので 1〜2 秒あける

### 未検証
- 取得・ループの流れは Commons API をモックしたヘッドレス Chromium で確認済み（段階の順、重複除外、一巡後の繰り返し）。実機での見た目の確認はまだ
- Commonsの検索語の当たり具合。`THEMES` の語は調整前提。アメリカと日本の街のテーマはコンタクトシートで上位7枚ずつ目視したが、40件以降やページ送り先は見ていない

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
