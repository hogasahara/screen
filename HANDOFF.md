# 引き継ぎ書：青の時間（チル写真スクリーンセーバー）

最終更新：2026-09-14（main = `c6d0e71`）

- リポジトリ：https://github.com/hogasahara/screen
- 公開 URL：https://hogasahara.github.io/screen/
- 作業ブランチ：`claude/wizardly-volta-0kgqns`（会話ごとに変わる。前回は `claude/loving-johnson-vqalf9`）。main に早送りでマージして push すると GitHub Actions が公開する。
  マージは本人の指示で行う（「確認は不要で main にそのまま反映」と言われた回もある。その都度の指示に従う）
- 会話は日本語。本人は実機で試して感想を返す。作成環境（Claude Code の遠隔コンテナ）には GPU も Unsplash の鍵も無い

## 目的と方向性

チルな音楽を流しながら、チルな写真をスクリーンセーバーのように眺めるための1枚もの Web ページ。
音楽は本人の手持ちアルバム。**写真の選びかたと見せかたが鍵**。

- 吉村和敏の写真集の雰囲気（ブルーモーメント、プリンス・エドワード島、雪の村と灯り、田園）と、
  スティーブン・ショア『Uncommon Places』の「何でもない場所」。
- 写真は保存せず、その場で取ってきて眺める。自分のサーバーに複製しない。
- 依存なし単一 HTML が基本方針。例外は雨の演出のライブラリ（同梱、必要なときだけ読み込む）。

### 本人がテーマごとに言葉にした「見たいもの」（語はここから決めている）
- ブルーモーメント：色あせたような色合いの、人々の営みを感じる写真。距離は近景から遠景まで
- プリンス・エドワード島：吉村和敏のテイスト。自然豊かで色彩も豊かな家や集落とその周辺。季節はすべて
- 雪と灯り：彩度の低い風景で、人々の営みが雪に包まれている。遠景が主体
- 田園と空：作物と共にある人々の営み。遠景が主体
- アンコモン・プレイス（アメリカ）：ショアの写真集の分析から。交差点と信号、駐車場と車、モーテルの外観と部屋、ガソリンスタンド、二車線道路と電柱・電線・看板、住宅街と木造の家、平屋の商業建築、ダイナーの食事、映画館の看板。日中の平らな光、褪せた色、中景〜遠景、水平、正面。人はいないか小さい
- 日本の街：都会以外の日本の家並みや店を、色あせたような色合いで。近景から遠景まで。昼も含めてよい
- マルタ島：本人の言葉は「地中海の観光地」だけ（2026-09-14、出先から）。上の方針を当てて、蜂蜜色の石灰岩の家並みと木の出窓、港と舟、集落と教会のドーム、近景から遠景まで、人はいないか小さい、とした。島全体が観光地なので「観光地の地名を避ける」方針の例外。首都（valletta）と漁港（marsaxlokk）は絵葉書に寄りやすいので差し込みの重みにしてある。**語はまだ本人の目で見ていない**
- 「人々の営み」は人物ではなく痕跡（灯りの窓、看板、耕された畑）。人はいないか小さい
- 「色あせた色合い」は表示側で彩度を落とすのではなく語で狙う（本人の判断。色あせた色調で撮る写真家はそれが映える被写体を選んでいる）。Unsplash では `film photography` を語に含める
- ヨーロッパの村は本人の判断で削除

## ファイル構成

| ファイル | 役割 |
| --- | --- |
| `index.html` | 本体。CSS・HTML・JS がすべて入った単一ファイル（約1100行） |
| `manifest.json`, `icon.svg`, `icon-192.png`, `icon-512.png` | ホーム画面に追加すると全画面で起動する PWA 設定とアイコン |
| `rain/raindrop-fx.js`, `rain/LICENSE`, `rain/README.md` | 雨の演出ライブラリ [raindrop-fx](https://github.com/SardineFish/raindrop-fx) 1.0.8 の同梱（MIT、SardineFish）。雨をオンにしたときだけ読み込む |
| `.github/workflows/pages.yml` | main への push で `index.html`・`manifest.json`・アイコン・`rain/` を GitHub Pages に配置 |
| `README.md` | 使いかた |
| `HANDOFF.md` | この文書 |

## いま出来ていること

### 写真の出どころ（設定パネルのラジオ）
1. **Wikimedia Commons**（鍵不要、CORS 可、`origin=*`）。既定
2. **Unsplash**（無料の Access Key が必要。開発モードは 50 要求/時。本人は鍵を持って試している。質が高い）
3. **Flickr**（コードはあるが、API キーは Pro アカウント限定になったので本人は使えない）
4. **お気に入りだけ**（登録した写真を繰り返す）
5. **自分のフォルダ**（端末内で完結）

鍵は URL のハッシュにだけ残り、リポジトリには入らない。

### テーマ（複数可）とキー
- ブルーモーメント／プリンス・エドワード島／雪と灯り／田園と空／アンコモン・プレイス（アメリカ）／日本の街／マルタ島
- キー：Space 停止・再開、→ 次、L お気に入り、R 雨の段階、F 全画面、C キャプション、D 検索語表示、S 設定、Esc 閉じる

### 見せかた
- 2枚のスライドを4秒のクロスフェード、ケン・バーンズ（雨のときは無し）
- 「青の深さ」：青系グラデーションを `mix-blend-mode: multiply` で重ねる（既定 28%）。スライダーの id は `tintRange`（かぶせの `#tint` と同じ id にしてバーが消えていた事故があった）
- フィルムグレイン、ビネット、左下キャプション（撮影者・ライセンス・出典、お気に入りは ♥）、右下の時計
- 操作しないとカーソルと HUD が消える（3.5秒）
- 設定は URL ハッシュに base64 JSON。お気に入りだけ `localStorage`（`aonojikan.favorites`）
- 「高度な設定」（`<details>`、既定で閉じる）：雫の陰影、窓の外のにじみ切り替え、検索語の表示

### お気に入り
- L か HUD で登録。JSON の書き出し・クリップボードへコピー・読み込み（追記）・全消去
- 記録：`{id, source, theme, query, tier, url, thumb, page, title, credit, author, savedAt}`。共有と語の調整に使う

### 検索語の当たりを見る（設定のテーマの下のボタン）
いまの出どころとテーマの語ごとに、通常検索の1ページ目をサムネイルで一覧にする。取ってあればそれを使う。写真を押すとお気に入りに入る。Unsplash では語の数だけ要求を使う。**語の評価は本人の目で行う**ための道具。

## 取得ロジック（`index.html` の「写真の在庫」）

- `THEMES[t].commons / .unsplash / .flickr` に語のリスト。語は文字列か `{q, w}`。`w` は出す比率の重み（主題 1、差し込み 0.2〜0.6）。`plain: true` は認定カテゴリを探さない。`deep` はその語だけ深く読む上限。`oneAuthorOk` は同じ写真家の上限を外す（Margolies 用）
- 取りに行く先は `feeds`（語 × 段階）。Commons の段階は 秀逸な画像（`incategory:"Featured pictures on Wikimedia Commons"`、上限200件）→ 品質画像（`incategory:"Quality images"`、400件）→ 通常検索（関連度順、80件。後ろほど外れる）。上の段階に残りがある限りそこから
- `feedCache`（出どころ×段階×語）に取った分を全部残す。テーマや出どころを切り替えても捨てない。Unsplash の要求を無駄にしない
- 各 feed は `buffer` と `overflow` を持つ。`admit()` が重複（id と、jpg/tif 二重登録のためのタイトル `dupKey`）を除き、同じ写真家（1語3枚・全体12枚）と同じ場所名（1語2枚）を超えた分を `overflow` へ。その段階のどの語にも buffer が無くなったら overflow を出す（捨てない）
- `refill()`：在庫が6枚を切ったら、いちばん上の段階で残りのある語をぜんぶ対象に、バッファが空の語を並列で取り（最多6語）、`pickWeighted()` で重みに比例して語を選びながら12枚ほど取ってシャッフル。直前と同じ語はほかに選べるなら避ける
- 表示した写真は `history` に残し、全 feed を使い切ったら `recycle()` がシャッフルして戻す（直近24枚は後ろ）。自分のフォルダとお気に入りも全部見せてから混ぜ直す
- Commons は `width >= 1600` かつ横長（比率1.15以上）だけ。`iiurlwidth=2400`。サムネは `/2400px-` → `/640px-`。Unsplash は `raw` に `w=2400`／`w=480`
- 写真には `source, theme, query, tierName, author, place, page` を持たせる
- 鍵が無い・断られたときは `halted` を立てて止める。鍵を入れ直すか出どころを変えると解ける
- `window.__aono()`：検索語の表示がオンのとき、雨まわりの内部状態を返す調整用の窓口

## 検索語について確かめたこと

- Commons の検索は語をすべて含むものだけ（AND）。語が多いほど外れる。`-tokyo` のような除外語が使える（API で確認済み）。名詞と場所名しか効かない
- Unsplash はタグと説明文に当たる。光と時間（dusk, blue hour, overcast）、距離（street, exterior, distant）、色調（film photography）の語が効く。除外語は使えない
- 認定カテゴリは風景には多いが絵葉書的な構図に偏り、「何でもない場所」には向かない（`plain`）
- 観光地の地名はテンプレ観光地を呼ぶ（銀山温泉、奈良井宿…は不採用）。**作家名は最強の絞り込み**。アメリカの Commons は John Margolies のロードサイド写真（議会図書館、約9800枚、パブリックドメイン）だけにした。200件目以降でも作品のまま。`sign` は看板の寄りばかりなので外した。Highsmith、テキサスの大通り、ダイナーは記録写真や古写真が混ざるので外した
- Commons のアマチュア記録写真は「何でもない場所」ではあっても作品としての構図と光の意図がない。本物さと雑さは別物。日本側は Commons では限界。Unsplash は写真家が作品として上げているので質が高い
- 語ごと均等に出すと踏切のように視覚的に強い被写体が目立つ。ヒット率で傾斜をつけるのは危うい（一般的な語ほど Unsplash 的なきれいな絵に寄る）。主題と差し込みの重み `w` で編集する。長い目ではお気に入りの蓄積で語と写真家を評価するのが本筋。Unsplash には写真家単位・コレクション単位の取得もある（未実装）
- 使用回数順（`gsrsort=incoming_links_desc`）は記事に載せやすい記録写真が上に来るので審美性とは無関係
- 日本の Commons 語は、シリーズで上げている投稿者の語（`danchi`、`japan residential street`、`"street view" fukuoka`、`japan port town street`、`japan level crossing street`、`incategory:"Snack bars in Japan"`）
- マルタ島（2026-09-15 に API で確認）：Commons の認定画像はマルタ全体で約550枚のひとつのプール（Diego Delso の 2021 年 8 月の連作と BW の 2011 年連作が大半）。`malta village` `malta street` `malta coast` `malta boats` はどれも品質画像 544〜585 件で、一般語は絞り込みに効いていない。効くのは地名：`gozo village`（品質 102、田園と集落の眺め）、`valletta`（71、街路）、`mdina`（45）、`marsaxlokk`（34、漁港）、`malta coast town`（16、スリーマの色つき出窓）、`grand harbour malta`（10、港と三姉妹都市）。`malta village` はポパイ村（映画のセットの遊園地）と語学学校の宿泊施設が上位に来るので使わない。`malta harbour` は認定に水中生物が多い。`malta church dome` はモスタのドームばかり。同じ写真家・同じ場所名の上限が連作をほぐす前提
- **Unsplash の新しい語の当たりはまだ本人の目で見ていない**（マルタ島も含む）。「検索語の当たりを見る」で確かめてもらうところで止まっている（雨の作業が先になった）

## 雨の窓ガラス（演出）

### 経緯
本人の要望：雨の日の窓越しの景色、ガラスをつたい落ちる水、その多寡（小雨〜豪雨）。音は後回し。
既存ライブラリを広く調べ、この環境で3つ動かして見比べた。

| 候補 | 方式 | ライセンス | 判断 |
| --- | --- | --- | --- |
| raindrop-fx（SardineFish） | WebGL2 | MIT | 採用。物理（衝突・合体・蒸発）と調整項目が豊富、1080p で数ms/フレーム |
| Codrops RainEffect（Lucas Bebber, 2015） | WebGL1 + Canvas | Codrops（組み込み自由） | 背景を毎フレーム更新できる代替候補。結合すれば23KB。項目は少ない |
| rainyday.js | Canvas 2D | GPLv2 | 品質低、ライセンスも合わない |
| Shadertoy「Heartfelt」系の自作 | シェーダー1枚 | 原作は CC BY-NC-SA で流用不可 | 考え方だけ借りて自作は可能。物理でなく格子の周期 |
| three.js 透過材質 | three.js | MIT | 静止した水滴模様で動かない |

### 仕組み
- 設定「雨」（`state.rain` 0〜100）。R で なし→小雨→雨→本降り→豪雨（0/15/40/70/100）。0 より大きいときだけ `rain/raindrop-fx.js` を読み込む。WebGL2 が無ければ案内して切る
- 雨のときは `#stage`（CSS のスライド）を隠し、`#rain` のキャンバスに raindrop-fx が写真ごと描く。写真は中間キャンバス `rainBg` に cover で描いて `setBackground(rainBg)` で渡す。写真が替わったら `rainShow(img)` が4秒かけて前の写真から溶け込ませる（100ms ごとに描き直して渡す。`setBackground` はミップマップ生成が要るので毎フレームには向かない）
- 雨のときはケン・バーンズをしない（本人の指示。切り替えで変形が飛んで不自然だった）
- WebGL のテクスチャに使うため、雨のときは `preload(url, true)` で `crossOrigin='anonymous'`。Commons と Unsplash は CORS 対応を確認済み。Flickr は未確認
- にじみ段階 0 だとライブラリは合成用の背景（`blurryBackground`）を作らないので、`blurBackground` を包んで `renderer.blit(background, blurryBackground)` を足している（同梱 1.0.8 固定なので動く）
- 霧（mist）は `false` にすると合成が滴のマスクだけになるので、完全に透明な霧を置いて見た目だけ消している
- 雨雲の下の暗さ：`#rainShade`（黒の層）を雨量に比例して不透明度 0→0.22
- 青の深さ・粒子・ビネットは雨の上にそのまま乗る

### 雨量 → raindrop-fx の設定（`rainOptions(v)`、現行値）
つまみ v（1〜100）を `t = ((v-1)/99)^1.6` に写す（真ん中は穏やか、右端で急に強く）。雫の大きさだけは `ts = min(1, t/0.45)`（雨量60あたりで頭打ち）。

| 項目 | 小雨(v=1) → 豪雨(v=100) | ねらい・備考 |
| --- | --- | --- |
| spawnInterval（生成間隔・秒） | [0.35, 0.9] → [0.004, 0.01] | 豪雨は毎フレーム級 |
| spawnSize（雫の大きさ） | [38, 75] → [45, 105]（ts） | 雨量60で頭打ち。以前の上限200は「雫が大きすぎる」と不評 |
| spawnLimit（雫の上限） | 250 → 5000 | 量。実機の負荷は未確認 |
| dropletsPerSeconds（小粒/秒） | 40 → 2500 | 画面が細かい粒で埋まる |
| dropletSize（小粒の大きさ） | [6, 16] → [8, 22] | 小さめ |
| slipRate（滑りやすさ） | 0.05 → 0.75 | 付いた雫がすぐ流れる |
| evaporate（質量の減り/秒） | 5 → 1 | 「蒸発が早い」への対応。既定10より遅い |
| trailDropDensity（跡の密度） | 0.2 固定 | 既定。「筋が太い」への対応で既定に戻した |
| trailDropSize（跡の滴の大きさ） | [0.3, 0.5] 固定 | 既定 |
| trailDistance（跡の間隔） | [20, 30] → [8, 14] | 間隔だけ詰めて細い筋が途切れない |
| xShifting（横風） | [0, 0.02] → [0, 0.25] | 斜めに流れる |
| gravity（重力） | 既定 2400（触らない） | 本人の指示 |
| colliderSize（合体のしやすさ） | 既定 1.0（触らない） | 本人の指示 |
| mist / mistColor / mistTime | true / [0,0,0,0] / 1e9 | 透明な霧。見た目は無し |
| backgroundBlurSteps / mistBlurStep | 0（「窓の外を少しにじませる」で 1） | 本人は段階1のにじみも許容しない。0 が既定 |
| refractBase / refractScale（屈折） | 0.55 / 0.9（にじみ1のとき 0.4 / 0.6） | 段階0では滴が背景に溶けやすいので強める |
| raindropDiffuseLight（拡散光） | sd = 0.12 + 0.23·sh | sh = 雫の陰影/100。既定73 → 0.288 |
| raindropShadowOffset（影の強さ） | 0.3 + 0.7·sh | 既定73 → 0.81。シェーダーは `color += (光の当たり具合 − 影の強さ) × 拡散光` なので高いほど滴が一様に暗くなる |
| raindropSpecularLight / Shininess | 0.25 / 128 固定 | ハイライトで輪郭を出す |
| `#rainShade` の不透明度 | 0 → 0.22（線形） | 雨雲の下の暗さ。ブラウザのフィルタは使わない |

### 実機フィードバックと対応の履歴
1. 曇りとぼかしは不要、雨中のケン・バーンズは不自然、蒸発が早い、豪雨が足りない → 霧を透明に、にじみ最小、ケン・バーンズ停止、蒸発を遅く、豪雨の数値を上げる、暗さの層
2. 段階1のにじみも許容できない、段階0を見たい → 段階0を既定に（`blurBackground` の差し替え）、屈折と光を強める、比較用の切り替えを高度な設定に
3. 豪雨で雫が大きすぎ筋が太すぎる → 大きさは雨量60で頭打ち（上限105）、跡の太さは既定に戻す、勢いは数と速さで
4. 影で雫が黒っぽく汚れて見える → 「雫の陰影」スライダー（影の強さと拡散光を連動）。本人が 73 を選び既定に。高度な設定へ

### 保留・未実装
- 水膜の揺らぎ層（ブラウザのフィルタで画面を歪ませる案）：本人の判断で保留
- 雨音：素材の調査は済み（下の「雨音の素材調査」）。実装と本人の試聴はこれから
- 雨のときに曇りや夜の写真へ寄せる語の連動：提案のみ
- 実機での豪雨の負荷（上限5000・小粒2500/秒）は未確認。重ければ上限から落とす

## 雨音の素材調査（2026-09-14）

本人の要望は「窓越しの雨音を、雨の多寡に合わせて」。作成環境では音を聞けないので、**最終の選別は本人の耳**。
候補集めはサブエージェントに任せ、表だけ受け取った（トークン節約のため、以後もこの形で）。

### ライセンスの判定（リポジトリに同梱して GitHub Pages で公開する前提）
| 出どころ | 判定 | 理由 |
| --- | --- | --- |
| Freesound の CC0 音源 | **可**。本命 | 再配布・改変・商用すべて可、表記不要。窓越しの実録音が豊富。プレビュー MP3（128kbps）の配信元 `cdn.freesound.org` は `Access-Control-Allow-Origin: *` を確認済みなので、同梱せず「その場で取る」も可能。本体（WAV）の取得はログインが要る |
| Wikimedia Commons の PD / CC0 | 可。予備 | `Rain against the window.ogg`（PD、1:22、mono 128k、英国の海辺、風強め）、`Urban Street on a Rainy Afternoon.flac`（CC0、30分、91MB、街の雨）。`upload.wikimedia.org` は CORS 可、鍵不要。ほかの雨音は CC BY-SA が多い |
| OpenGameArt「Rain (loopable)」Ylmir | 可 | CC0、窓で録った 25〜45 秒のループ 4 本、MP3/OGG。mono を疑似ステレオ化 |
| OtoLogic | 表記すれば可 | CC BY 4.0。CC0 で足りなければ |
| 効果音ラボ | **不可** | 商用・表記不要だが「ユーザーがダウンロード可能な状態で置く」ことと再配布を禁止。GitHub 上に置くこと自体が当たる |
| Pixabay | 避ける | 単体での再配布を禁止。リポジトリに素材ファイルを置くのはグレー |
| BBC Sound Effects（RemArc） | 不可 | 個人・教育・研究に限る。公開ページに置けない |
| SFXMint | 避ける | CC0 だが AI 生成、10 秒前後でループの継ぎ目なし |
| 魔王魂 | 避ける | 単品の再配布禁止、表記が要る |

### Freesound CC0 の候補（すべて各ページでライセンスを確認済み。窓越しの室内録音、雷・声・車なし）
| 強さ | id | 題名（作者） | 長さ | 元形式 | 備考 |
| --- | --- | --- | --- | --- | --- |
| 小 | 648529 | RAIN on glass window（nicoproson） | 2:54 | WAV 96k | **推し**。長くてループ向き |
| 小 | 473555 | Light Rain Recorded from Inside a Shut Window 2（timothyd4y） | 0:58 | WAV 48k | |
| 小 | 669486 | Rain on window (interior)（xkeril） | 0:58 | WAV 48k | 強弱の変化あり |
| 小 | 333510 | January rain on a window（mmorgaine） | 0:45 | AIFF 48k | 天窓。短い |
| 中 | 869851 | Rain_Hitting_Window_9（SignatureSoundsOrg） | 1:03 | WAV 44.1k | **推し**。同作者のパック「Rain Hitting Window」に姉妹音源 |
| 中 | 574673 | Rain on window PEI summer 03（TRP） | 1:04 | WAV 48k | プリンス・エドワード島の夏の雨 |
| 中 | 577305 | Rain, skylight window, interior, 2011（TRP） | 0:47 | WAV 48k | 詳細未取得 |
| 中 | 428605 | Rain on Metal Window Ledge（Erbsland-Music） | 1:44 | AIFF 44.1k | 風と街の音がわずかに入ると説明にある。要試聴 |
| 中 | 81819 | Rain on Window, Reverberant room（silencyo） | 0:49 | AIFF 48k | 撮影用の人工雨。避ける |
| 大 | 587000 | Heavy rain outside window（FrostCP） | 1:00 | WAV 44.1k | **推し**。夜、5 本のマイク |
| 大 | 672694 | Window heavy rain（Cinetony） | 2:33 | WAV 48k | 台所の窓。長い |
| 大 | 577298 | Rain, heavy on skylight window, interior, 2011（TRP） | 1:37 | WAV 48k | 天窓 |
| 大 | 243781 | rain against window 2（bastipictures） | 1:05 | MP3 320k | 小窓に強い雨 |
| 大 | 855890 | Heavy rain from inside, closed window（Chris.sonido.peru） | 2:39 | WAV 48k | 末尾に椅子の音と説明にある |

プレビュー URL の形は `https://cdn.freesound.org/previews/<id の先頭3桁>/<id>_<投稿者id>-hq.mp3`（例：`previews/648/648529_457982-hq.mp3`、3.8MB）。投稿者 id はサウンドページの HTML から取れる。
TRP はトロントの録音家で、窓・天窓の CC0 シリーズが多い（715609、567108、575260、574861、717572、717555 なども）。

### 実装の案（未着手）
- 雨量（`state.rain`）に合わせて 小・中・大 の 3 本を Web Audio の gain でクロスフェード。ループの継ぎ目は頭と尻を 1〜2 秒重ねる
- 音源の置き場は二択。(a) CC0 の本体を `rain/` に同梱（OGG/MP3 に圧縮、3 本で 5MB 前後。`rain/LICENSE` に出典を併記）。(b) Freesound のプレビュー URL をその場で取る（写真と同じ思想。鍵不要、CORS 可。ただし URL の形が変わる可能性と、128kbps の質）
- 選別は本人の耳が要るので、設定に「雨音の候補を聴く」を作り、上の表の候補を並べて試聴・採用できるようにする（写真の「検索語の当たりを見る」と同じ発想）。採用した id を設定に残す
- 音楽（本人のアルバム）と同時に鳴るので、雨音の音量つまみは別に持つ

## 検証の方法（作成環境でできること）

- 実 API は curl で叩ける。Commons は連続で叩くと "too many requests" になるので 2 秒以上あける
- ブラウザ（ヘッドレス Chromium `/opt/pw-browsers/chromium`）は外部 TLS が通らない（プロキシの CA を信用しない）。`fetch` をモックし、写真はローカルの jpg を `python3 -m http.server` で配る。`file://` では画像の CORS と `fetch` が失敗する
- Commons のモックは応答の `width` を 1600 以上に偽装しないとフィルタで捨てられる
- WebGL2 は `--use-gl=angle --use-angle=swiftshader --enable-unsafe-swiftshader` で動く（CPU 描画で遅い。仮想時間 `--virtual-time-budget` では滴が育たない）。実時間で 25 秒動かし、ページ側で `requestAnimationFrame` 内に `canvas.toDataURL()` して POST で受け取る（WebGL のキャンバスは描画直後でないと読めない）
- コンタクトシート：API でサムネを取って HTML に並べ、スクリーンショットにすると語の当たりを目で確かめられる

## その先の候補
- Unsplash の写真家単位・コレクション単位の取得（お気に入りの `author` から辿る）
- お気に入り・除外の蓄積で語と写真家に重みをつける
- 時間帯でテーマや青の深さを自動で変える
- 雨音、雷、雪、車窓（アンコモン・プレイス向き）

## 注意
- Unsplash・Flickr の鍵をコードやコミットに入れない（URL ハッシュにだけ残る設計）
- Commons の写真は CC 系。キャプションの撮影者・ライセンス表示は消さない。Margolies はパブリックドメイン
- Unsplash は表示時に撮影者名を出す規約。キャプションを消せる設定はあるが既定では出す
- `rain/raindrop-fx.js` は MIT（SardineFish）。`rain/LICENSE` を一緒に配る
