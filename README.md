# 美容師のためのAI実践講習 申込LP

広告（クリエイティブA・B・C）・Instagram・DM からの申込を受けるページ。主催：株式会社Beauty Grow。
デザインは広告バナー（クリーム地・濃い緑の明朝見出し・サーモンの下線・赤の「参加無料」）に合わせている。

## 0. 広告3本ごとのURL（ファーストビューがバナーと同じ見出しに切り替わる）

`utm_content` の値で、見出し・リード文・スタッフの声・タグが変わる。文言は index.html の `VARIANTS` で直す。値がない・違うときはA案の表示になる。

```
A 閉店後のPC作業     https://kakiuchi-ops.github.io/ai-event-lp/?utm_source=meta&utm_medium=paid&utm_campaign=ai1201&utm_content=a
B 口コミ返信・投稿文 https://kakiuchi-ops.github.io/ai-event-lp/?utm_source=meta&utm_medium=paid&utm_campaign=ai1201&utm_content=b
C 店長の事務仕事     https://kakiuchi-ops.github.io/ai-event-lp/?utm_source=meta&utm_medium=paid&utm_campaign=ai1201&utm_content=c
```

- スプレッドシートの「utm_content」列・担当者あての通知メールで、どの広告から来た申込かがわかる
- **DMからの申込は広告別に数えられない。** DMで来た人には「どの投稿を見たか」を聞いて、シートに手で追記する
- ※URLは公開先が決まったら差し替える（個人名の入らないURLにする場合は joukyo-recruit-lp と同じ方法）

## 残り席数の表示
index.html の CONFIG `seatsLeft` に数字を入れると、ファーストビューに「残り◯席」が出る（0なら「満席（キャンセル待ちで受付中）」）。`null` の間は表示しない。シートの「受付」件数を見て手で更新して push する。

## 申込の受け取り方
1. **フォーム** → GAS → スプレッドシートに記録＋担当者のメール（スクリプトプロパティ `NOTIFY_TO`）に全項目を送信＋申込者に自動返信。Chatwork通知は任意
2. **InstagramのDM** → index.html の `instagramId` に採用インスタのID（@なし）を入れると、「InstagramのDMで申し込む」ボタンと主催者欄のInstagramが表示される。入れるまでは非表示

```
index.html    LP本体（CSS・JSも同じファイル内）
thanks.html   申込完了ページ（Metaピクセル PageView＋Lead）
privacy.html  プライバシーポリシー（仮。正式版に差し替える）
images/       ogp.png(1200×630 ※仮) / instructor-saito.png / staff-takahashi.png / staff-kato.jpg / staff-yajima.png（hero.png・stylist.png は未使用）
gas/Code.gs   申込の受け口（GitHubには載せない。.gitignore で除外済み）
```

---

## 1. 【 】の差し込み一覧

ファイル内を「【」で検索すると見つかります。

| 項目 | ファイル | 確認先 |
|---|---|---|
| 申込締切 11月30日（月）（仮で設定） | index.html（開催概要の表・CONFIG の `deadline`）・gas/Code.gs | 変えるときは3か所そろえる |
| 【ピクセルID】 | index.html・thanks.html | Meta広告の担当者 |
| 【GASのURL】 | index.html の CONFIG | 手順3で発行 |
| テストモードのスイッチ `window.LP_TEST_MODE` | index.html の `<head>` 先頭 | 公開前は `true`、告知開始前に `false`（手順4） |
| 【公開URL】（og:url・og:image） | index.html | 手順4で決まる `https://kakiuchi-ops.github.io/ai-event-lp` |
| 【問い合わせ先メールアドレス】 | privacy.html・gas/Code.gs | 総務／運営チーム |
| 【会社所在地】【制定日】 | privacy.html | 総務（正式版に差し替えるなら不要） |

※ gas/Code.gs のメール件名にある「【お申し込み受付】」などは件名の飾りです。差し込み項目ではないので、そのまま残します。
※ ピクセルIDとGASのURLが【 】のままだと、ピクセルは動かず、フォームは「送信先の設定中です」と表示します（壊れた状態で公開されないための安全装置）。

---

## 2. ローカルでの表示確認

ターミナルで次をコピペします。

```
cd /Users/kakina/cockpits/beautygrow/lp/ai-event-lp
python3 -m http.server 8765
```

ブラウザで次を開きます（止めるときはターミナルで `Ctrl + C`）。

```
http://localhost:8765/?utm_source=test&utm_medium=local&utm_campaign=check
```

- **スマホ表示**：Chrome で `⌘ + Option + I` → 左上のスマホ型アイコン（`⌘ + Shift + M`）→ 機種で「iPhone 12 Pro」などを選ぶ
- **スマホ実機**：Mac とスマホを同じWi-Fiにつなぎ、`ipconfig getifaddr en0` で出たIPを使って `http://<IP>:8765/` を開く
- **確認すること**：下の固定ボタンがフォームまで来ると隠れる／未入力で送信すると赤いエラーが出る／thanks.html が表示できる

---

## 3. GAS（申込の受け口）のセットアップ

### 3-1. スプレッドシートとコード
1. Googleドライブで新しいスプレッドシートを作成（名前の例：`AI実践講習 申込`）
2. メニュー「拡張機能」→「Apps Script」
3. 最初からある `function myFunction() {}` を全部消し、`gas/Code.gs` の中身を全部貼り付けて保存（`⌘ + S`）
4. 上部の関数選択で `setup` を選び「実行」
   - 初回は「承認が必要です」→ 自分のアカウントを選ぶ →「詳細」→「（安全ではないページ）に移動」→「許可」
   - スプレッドシートに「申込」シートと見出し行ができる

### 3-2. スクリプトプロパティ（APIトークンなどの保管場所）
左の歯車「プロジェクトの設定」→ 一番下「スクリプト プロパティ」→「スクリプト プロパティを追加」

| プロパティ | 値 |
|---|---|
| `NOTIFY_TO` | 申込を受け取る担当者のメールアドレス（複数ならカンマ区切り） |
| `CHATWORK_API_TOKEN` | （任意）Chatwork の APIトークン（Chatwork右上の自分の名前 →「サービス連携」→「APIトークン」） |
| `CHATWORK_ROOM_ID` | 通知先ルームのID（ルームを開いたURLの `#!rid` の後ろの数字） |
| `REPLY_TO` | （任意）自動返信メールに返信されたときの宛先 |

トークンはコードに書かない・チャットやメールに貼らないでください。

### 3-3. 動作テスト（デプロイ前）
1. 関数選択で `testDoPost` を選び「実行」
2. 確認すること：スプレッドシートにテスト行が入る／Chatwork に通知が来る／自分宛てに自動返信メールが届く
3. テスト行はステータス「テスト」で記録され、定員のカウントには入らない（不要なら削除してOK）

### 3-4. ウェブアプリとしてデプロイ
1. 右上「デプロイ」→「新しいデプロイ」
2. 歯車「種類の選択」→「ウェブアプリ」
3. 設定
   - 説明：`AI講座 申込 v1`
   - **次のユーザーとして実行：自分**（スプレッドシートへの書き込みとメール送信を、あなたの権限で行うため）
   - **アクセスできるユーザー：全員**（Googleにログインしていない申込者も送信できるようにするため。「Googleアカウントを持つ全員」では送信が失敗します）
4. 「デプロイ」→ 表示された **ウェブアプリのURL**（`https://script.google.com/macros/s/…/exec`）をコピー
5. index.html の `endpoint: '【GASのURL】'` をこのURLに置き換える

**コードを直したとき**は「デプロイ」→「デプロイを管理」→ 鉛筆アイコン → バージョン「新バージョン」→「デプロイ」。こうするとURLは変わりません（「新しいデプロイ」を選ぶとURLが変わるので注意）。

### 受け口でやっていること
- 見えない入力欄（honeypot）に入力があれば記録しない
- 必須項目・メール・電話番号・立場（店長／スタイリスト）を再チェック
- 先着10名を「受付」、11人目以降を「キャンセル待ち」、同じメールアドレスの再送を「重複」、締切後を「締切後」として記録する（定員を変えるときは Code.gs の `capacity`）
- 「=」「+」「-」「@」で始まる入力は文字列として保存する（数式インジェクション対策）
- Chatwork 通知には、メールアドレスと電話番号を載せない（詳細はシートで確認）
- 申込者が手動でキャンセルしたときは、シートのステータスを「キャンセル」に書き換えれば、その分の枠が空きます

---

## 4. テストモード（公開前の確認）

index.html の `<head>` の先頭にスイッチがあります。

```
window.LP_TEST_MODE = true;   // テスト中
window.LP_TEST_MODE = false;  // 本番
```

`true` の間は次のようになります。
- 画面上部に黄色の「テスト中のページです」帯が出る
- 検索エンジンに載らない（noindex）
- 申込はスプレッドシートにステータス「テスト」で記録され、**定員10名のカウントに入らない**
- Chatwork 通知とメール件名の先頭に `[TEST]` が付く
- Metaピクセルは本番と同じく動く（テストイベント画面で確認するため）

### テストの流れ
1. 手順3でGASを用意し、index.html に GASのURL・ピクセルID・公開URL を入れる（`LP_TEST_MODE = true` のまま）
2. 手順5で GitHub に push して Pages を有効にする
3. iPhone・Android の実機で `https://kakiuchi-ops.github.io/ai-event-lp/?utm_source=test&utm_medium=test&utm_campaign=test` から申し込む
   - スプレッドシートに「テスト」行が入る／Chatwork に `[TEST]` 通知が来る／`[TEST]` 件名の自動返信メールが届く
4. Metaイベントマネージャ →（データセット）→「テストイベント」で、公開URLを開いて PageView、申込後に PageView＋Lead が届くことを確認
5. シェアデバッガー https://developers.facebook.com/tools/debug/ に公開URLを入れ、OGP画像とタイトルを確認
6. 公開前チェックリスト（手順6）を全部確認

### 本番に切り替える
1. index.html の `window.LP_TEST_MODE = true;` を `false` に変える
2. 次をコピペして push（1〜2分で反映）
```
cd /Users/kakina/cockpits/beautygrow/lp/ai-event-lp
git add index.html
git commit -m "Go live"
git push
```
3. スマホで公開URLを開き、黄色の帯が消えていることを確認してから告知を始める
4. スプレッドシートの「テスト」行は、残しても定員の判定には影響しない（見づらければ削除）

---

## 5. GitHub Pages への公開

### 4-1. GitHubでリポジトリを作る（ブラウザ）
1. https://github.com/new を開く（kakiuchi-ops でログイン）
2. Repository name：`ai-event-lp`／**Public**／README などのチェックは全部外す →「Create repository」

### 4-2. push（ターミナルにコピペ）
```
cd /Users/kakina/cockpits/beautygrow/lp/ai-event-lp
git init
git add .
git status
```
`git status` の一覧に **`gas/` が出ていないこと**を確認してから続けます。

```
git commit -m "Add AI event LP"
git branch -M main
git remote add origin https://github.com/kakiuchi-ops/ai-event-lp.git
git push -u origin main
```

### 4-3. Pages を有効にする
1. リポジトリの「Settings」→ 左メニュー「Pages」
2. Source：「Deploy from a branch」／Branch：`main`・`/(root)` →「Save」
3. 1〜2分で `https://kakiuchi-ops.github.io/ai-event-lp/` が公開される

### 4-4. 修正したとき
```
cd /Users/kakina/cockpits/beautygrow/lp/ai-event-lp
git add .
git commit -m "Update event info"
git push
```

### 経路別URL（広告・投稿・LINE・DMで使い分ける）
```
Meta広告   https://kakiuchi-ops.github.io/ai-event-lp/?utm_source=meta&utm_medium=paid&utm_campaign=ai1201
Instagram  https://kakiuchi-ops.github.io/ai-event-lp/?utm_source=instagram&utm_medium=organic&utm_campaign=ai1201
公式LINE   https://kakiuchi-ops.github.io/ai-event-lp/?utm_source=line&utm_medium=message&utm_campaign=ai1201
DM         https://kakiuchi-ops.github.io/ai-event-lp/?utm_source=dm&utm_medium=direct&utm_campaign=ai1201
```

---

## 6. 公開前チェックリスト

**内容**
- [ ] 赤字の「※社内確認用」3か所（`hero-provisional`・`cases-provisional`・`comment-provisional`）を、差し替えたうえで削除した
- [ ] ファーストビューのスタッフの声（A・B・C）が、実在のスタッフの実際の発言で、掲載の同意がある（架空の体験談は景品表示法・Meta広告ポリシーの問題になる。バナー側も同じ）
- [ ] 高橋さん・加藤さん・矢島さんの仮コメントを本人の言葉に差し替え、写真の掲載同意を取った（加藤さんの写真はHPBから取得）
- [ ] 齋藤さんのコメントを本人の言葉に差し替えた
- [ ] ファーストビューの画面イメージ（口コミ返信・振り返りの例）が、講習で実際に見せる内容と合っている
- [ ] 「参加無料」の近くに必ずClaude Proプラン自己負担の注記がある（LP・バナー・投稿文すべて）
- [ ] 【 】が残っていない（「【」で検索。メール件名の飾りは除く）
- [ ] 日付と曜日が一致（12/1は火曜、11/30は月曜。変更したらカレンダーで再確認）
- [ ] 表示用の締切日と、CONFIG・Code.gs の締切日時が同じ
- [ ] 定員10名が LP・Code.gs（`capacity`）・告知物で一致
- [ ] 講師名は「齋藤」表記（「斎藤」ではない）
- [ ] 主催：株式会社Beauty Grow の表記がある（ヘッダー・開催概要・フッター）
- [ ] OGP画像（images/ogp.png は仮）をバナーに合わせて作り直した
- [ ] privacy.html を正式版に差し替えた（またはこの仮版で公開してよいと確認した）

**フォーム・GAS**
- [ ] テストモードで、公開URLから実際に申し込み → シート（テスト行）・Chatwork・自動返信メールを確認
- [ ] スマホ実機（iPhone／Android）で申込みまで通る
- [ ] 自動返信メールが迷惑メールに入っていない

**Metaピクセル・広告**
- [ ] Meta の「テストイベント」で、index で PageView、thanks で PageView＋Lead が届いている
- [ ] thanks.html を再読み込みしても Lead が重複しない
- [ ] イベントの詳細に、氏名・メール・電話番号が載っていない
- [ ] 広告の特別広告カテゴリ（採用目的のため「雇用」に当たるか）を確認した
- [ ] SNSシェア時の表示を確認（https://developers.facebook.com/tools/debug/ に公開URLを入れる）

**本番切り替え**
- [ ] `window.LP_TEST_MODE = false;` にして push した
- [ ] スマホで黄色の「テスト中」帯が消えている
- [ ] 本番切り替え後に1件申し込み、ステータスが「受付」になる（確認後、その行はステータスを「キャンセル」に変えて枠を空ける）
