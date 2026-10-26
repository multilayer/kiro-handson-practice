# STEP 10 上級編：AI-DLC ワークフローで1本通す（準備45分＋実践105分）

## この回の目的

- AWS が公開している **AI-DLC ワークフロー**（[awslabs/aidlc-workflows](https://github.com/awslabs/aidlc-workflows)）を Kiro IDE に入れて動かせるようになる
- AI が進行役になり、**「質問 → 文書 → 人の承認」** を段階ごとに繰り返す進め方を体験する
- STEP 6〜7 の Spec と比べて、どんな仕事に向いているかを自分の言葉で言えるようになる

> STEP 1〜9 を終えた人向けの上級編です。ターミナル（PowerShell）でのコマンド入力があります。
> この回は **上位のモデル（Claude Opus 5.5）を使うため、Kiro の有料プランが必要で、クレジットの消費も大きく** なります。月の残りクレジットを確認してから始めてください（下の「クレジットの注意」）。

## まず違いを押さえる

| | Spec（STEP 6〜7） | AI-DLC ワークフロー（この回） |
|---|---|---|
| 進め方 | 要件 → 設計 → タスク → 実装 の3つの文書 | 5つのフェーズ・33の段階（ステージ）から、仕事に合う段階だけを選んで進む |
| 進行役 | 人が「次へ」を指示する | **AI が進行役**。段階ごとに質問し、文書を作り、承認を求める |
| 人の役目 | 文書を確認して直してもらう | 質問に答える／段階の終わりに **承認（Approve）か差し戻し（Request Changes）** を選ぶ |
| 記録 | `.kiro/specs/` の3つの文書 | `aidlc/` フォルダに、段階ごとの文書・質問と答え・承認の記録（監査ログ）が残る |
| 準備 | Kiro だけで使える | インストールと設定が必要（この回の前半） |
| 向いている仕事 | 小さな機能・ツール | 関係者が多い・記録を残したい・大きめの開発 |

### 5つのフェーズ

| フェーズ | ひとことで | この回で通る段階（poc の場合） |
|---|---|---|
| 0. 初期化（Initialization） | 作業記録のフォルダを作る | 自動で終わる |
| 1. 構想（Ideation） | 何を・なぜ作るかを決める | **意図の把握**（Intent Capture） |
| 2. 方向づけ（Inception） | 要件を詰め、設計する | **要件分析**（Requirements Analysis） |
| 3. 構築（Construction） | 作って試す | **コード生成**（Code Generation）、**ビルドとテスト**（Build and Test） |
| 4. 運用（Operation） | 動かし、ふりかえる | 通らない |

### この回で使う「ワークフローの型（プロファイル）」

AI-DLC には、仕事の種類に合わせた型が11種類あります。この回は一番短い **`poc`（試作）** を使います。

| 型 | 向いている仕事 | 人が答える段階の数 |
|---|---|---|
| **poc** | 「できるかどうか」を早く確かめる試作 | **約5**（この回） |
| express | 要件がはっきりした小さな変更 | 約7 |
| bugfix | 原因が分かっている不具合の修正 | 約6 |
| mvp | 最初の本番の版を作る | 約20 |
| feature | 本番の機能を、全工程で作る | 30 |

> 型を指定しないと、標準の `classic`（15段階）などが選ばれ、時間とクレジットを大きく使います。**この回は必ず `poc` と指定します。**

---

## 準備するもの

- STEP 2 でクローンしたハンズオン用リポジトリ（`C:\Users\＜自分＞\kiro\kiro-handson-practice`）。最新の main を **Pull** しておく
- Kiro IDE（バージョン **1.1.70 以降**。メニューの **Help → About** で確認）
- Kiro の有料プラン（Pro 以上）と、今月の残りクレジット（下の「クレジットの目安」）
- [お題のフォルダ](../exercises/step10-aidlc/README.md) を読み、「作業を始める前に考えておくこと」に目を通しておく

### クレジットの注意

- AI-DLC は、段階ごとに「質問 → 文書作成 → レビュー → 承認」を丁寧に繰り返すため、**Spec より多くのクレジットを使います**
- AI-DLC の公式は、**上位の Opus のモデル** を使うよう案内しています（弱いモデルだと、承認を待たずに進んでしまうことがある）。このハンズオンでは **Claude Opus 5.5**（消費は Auto の 2倍：モデル一覧の「2x Credit」）を使います
- Claude Opus 5.5 は「Experimental preview（試験提供）」です。一覧にない・動きがおかしいときは、**Claude Opus 5**（2.2倍）を使います
- 無料プランでは Opus のモデルを選べません。有料プランに切り替えてから行ってください
- この回が終わったら、モデルを **Auto に戻します**

### クレジットの目安

1本でどれだけ使うかは、モデルや答え方で大きく変わります。次のようにして使いすぎを防ぎます。

1. 始める前に残りクレジットを確認し、**月の半分以上**（Pro なら 500 以上）残っている月に行う
2. 段階1（意図の把握）を承認した時点で、もう一度残りを確認する
3. 段階1だけで、始める前の残りの **5分の1以上** を使っていたら、`/aidlc park` で止め、翌月や追加のクレジットで続ける（残りの3段階で足りなくなるおそれがあるため）
4. 終わったら、1本で使った量をメモしておく（次に試すときの目安になります）

> この目安は使いすぎを防ぐための仮のものです。実際の消費量はモデルや答え方で変わります。

---

## 前半：準備（45分）

### 1. 自分専用の作業フォルダを作る（5分）

AI-DLC は、入れたフォルダの中に自分用のファイルをたくさん作ります。ハンズオン用リポジトリとは **別のフォルダ** で試します。

1. スタートメニューで **「PowerShell」** と入力し、**Windows PowerShell** を開く（「管理者として実行」は **しない**）
2. 次のコマンドを1行ずつ貼り付けて Enter を押す（`taro` は自分の名前に）

   ```powershell
   cd $HOME\kiro
   New-Item -ItemType Directory -Force aidlc-taro
   Copy-Item kiro-handson-practice\exercises\step10-aidlc\* aidlc-taro
   Copy-Item kiro-handson-practice\materials\変更依頼一覧_サンプル.csv aidlc-taro
   ```

3. 中身を確認する

   ```powershell
   Get-ChildItem aidlc-taro
   ```

   `README.md` `technical-environment.md` `vision.md` `変更依頼一覧_サンプル.csv` の4つが出れば OK

### 2. AI-DLC をインストールする（10分）

**Kiro を閉じてから** 行います（Kiro が開いたままだと、インストール後に Kiro が `aidlc` を見つけられません）。

1. Kiro IDE を **完全に終了** する（右上の × で閉じる）
2. 同じ PowerShell で、次を **まとめて貼り付けて** Enter を押す

   ```powershell
   $download = Join-Path $env:TEMP "aidlc-install-$PID"
   New-Item -ItemType Directory -Force $download | Out-Null
   $installer = Join-Path $download install.ps1
   Invoke-WebRequest `
     -Uri https://github.com/awslabs/aidlc-workflows/releases/latest/download/install.ps1 `
     -OutFile $installer
   & $installer
   Remove-Item -Recurse -Force $download
   ```

3. `Installed` で始まる行（バージョンとインストール先）が出れば成功です
   - インストール先は自分のユーザーの中（`%LOCALAPPDATA%\aidlc`）なので、**管理者権限は要りません**
4. PowerShell を **いったん閉じて、開き直す**
5. 次を入力し、バージョンが表示されることを確認する

   ```powershell
   aidlc version
   ```

> 「このシステムではスクリプトの実行が無効になっている」と出たら、PC の設定で PowerShell スクリプトが止められています。[困ったとき集](troubleshooting.md) の「インストールで『スクリプトの実行が無効』と出る」を見てください。

### 3. 作業フォルダに AI-DLC を設定する（10分）

1. PowerShell で作業フォルダに移動する

   ```powershell
   cd $HOME\kiro\aidlc-taro
   ```

2. Kiro IDE 用に設定する

   ```powershell
   aidlc config --harness kiro-ide
   ```

3. `Initialize AI-DLC in C:\Users\＜自分＞\kiro\aidlc-taro? [y/N]:` と聞かれたら、**`y`** を入力して Enter
4. 最後に `Setup check` の表が出ます。**`0 of 8 sections need you`** と出て、すべての行が **`[ok]`** なら OK です

   ```text
     Setup check - 0 of 8 sections need you.

       [ok]     Harnesses   kiro-ide recorded
       [ok]     Models      every agent uses your Kiro session's model ...
       ...
       [ok]     Workspace   workspace shell present
   ```

   - 表の上に英語で「次は Kiro IDE でこのフォルダを開き、Trust して、Reload Window し、aidlc エージェントを選んで `/aidlc --doctor` を実行して」という案内が出ます。この手順は下の 4・5 で行います
   - `[ok]` 以外の行がある場合や、途中で質問が出た場合は、**表示された おすすめ（Enter）** のまま進め、最後の表の `[ok]` 以外の行を Kiro のチャットに貼って「どうすればよいですか」と聞いてください
   - 念のため `aidlc doctor` を実行して確かめることもできます。最後の行が **`0 problems`** なら OK です。次の2つの warning（注意）は、この時点では出て当然なので気にしません
     - `Update: update refresh unavailable; release metadata could not be checked`（新しい版の確認ができなかった。ネットワークの都合で出ることがある）
     - `AIDLC hooks have not run in this project yet`（このフォルダでまだ一度もチャットを送っていない）
5. フォルダの中に `.kiro` `aidlc` `AGENTS.md` などが増えたことを確認する

   ```powershell
   Get-ChildItem -Force
   ```

### 4. Kiro でフォルダを開き、信頼する（10分）

1. Kiro IDE を起動 → **「File」→「Open Folder...」** → `C:\Users\＜自分＞\kiro\aidlc-taro` を選ぶ
2. 「このフォルダの作成者を信頼しますか？」と聞かれたら **「Trust Folder & Continue」**（または「Yes, I trust the authors」）を選ぶ
   - 画面上部に **Restricted Mode**（制限モード）の帯が出ている場合は、帯の **Manage** → **Trust** を押す
     - 日本語表示の場合は、帯の「制限モードは、安全なコード参照のためのものです。…」の右にある **「管理」** を押し、開いた「ワークスペースの信頼」画面の左側「信頼済みフォルダー内」の **「信頼する」**（紫のボタン）を押す
   - 自分で作った、中身の分かっているフォルダなので信頼して大丈夫です。**人からもらったフォルダは、中身を確認するまで信頼しない** のが原則です
3. **`Ctrl` + `Shift` + `P`** を押し、`Reload Window` と入力して **「Developer: Reload Window」**（日本語表示では **「開発者: ウィンドウの再読み込み」**）を選ぶ

### 5. aidlc エージェントを選び、動作確認する（10分）

1. **`Ctrl` + `L`** でチャット欄を開く
2. チャット入力欄の右下（Autopilot スイッチの左）にある **「Default ⌄」**（エージェントの選択）を押し、**aidlc** を選ぶ
   - 一覧の上のほうは **BUILT-IN**（Default・Spec・Quick Spec・Bug Fix・Plan）です。ここからは選びません
   - 一覧を **下にスクロール** すると **WORKSPACE** という見出しがあり、その一番上の **「aidlc」**（名前が aidlc だけのもの）を選ぶ
   - その下の `aidlc-architect-agent` などは、AI-DLC が裏で使う担当役なので選びません
   - 選ぶと、右下の表示が「Default」から「aidlc」に変わります
   - 新しいチャットを開くたびに Default に戻ります。**毎回 aidlc を選び直す** のを忘れずに
3. チャット欄の下の **モデル選択**（最初は **Auto**）を押し、**Claude Opus 5.5** を選ぶ
   - 一覧の右側の「2x Credit」などは、Auto と比べたクレジットの消費の倍率です
   - Claude Opus 5.5 が一覧にない場合は **Claude Opus 5** を選ぶ
4. 右下の **Autopilot** スイッチを **オン** にする
   - オフ（Supervised）だと、AI-DLC がファイルを作るたびに「Review changes」の確認が出て、操作が大幅に増えます
   - オンにしても、**AI-DLC の質問と承認は必ず止まって待ってくれます**
5. チャットに次を入力して Enter

   ```
   /aidlc --doctor
   ```

6. 問題（problems）が 0 件と表示されれば準備完了です

---

## 後半：実践（105分）

### 6. ワークフローを始める（5分）

新しいチャットを開き、**エージェントが aidlc になっていることを確認してから**、次を送ります。

```
/aidlc poc --learnings off ./vision.md と ./technical-environment.md を読んで、書かれているものを作ってください。質問・文書・画面の文言はすべて日本語でお願いします。
```

| 部分 | 意味 |
|---|---|
| `/aidlc` | AI-DLC を始める |
| `poc` | 試作の型（短い型）を使う |
| `--learnings off` | 段階ごとの「次回に向けて覚えておくことは？」の質問を省く（時間とクレジットの節約） |
| `./vision.md` … | 何を作るかは、このファイルを読んでもらう |

1. AI-DLC が型の確認をしてきます（例：「poc で 8 段階、承認 ○ 回」）。**`poc` のまま進める** 選択肢の番号を入力する
2. 初期化の3段階は自動で終わり、`aidlc/spaces/default/intents/＜日付＞-＜名前＞/` という作業記録のフォルダができます

> 選択肢や一部の表示は **英語のまま** 出ます。よく出る英語は下の「よく出る英語」の表を見てください。
> 選択肢には番号が付いています。**番号（`1` など）を入力して Enter** で答えます。

### 7. 段階1：意図の把握（Intent Capture）（20分）

1. AI-DLC が質問の一覧をファイル（`intent-capture-questions.md`）に書き出し、**答え方** を聞いてきます

   ```
   1. Guide me           … ここで1問ずつ答えていく
   2. I'll edit the file … ファイルに直接答えを書き込む
   3. Chat               … 自由に話し、AI に決定事項をまとめてもらう
   4. Other              … 別の方法を書く
   ```

2. 今回は **`1`（Guide me）** を選ぶ
3. 質問に答える。迷ったら [お題](../exercises/step10-aidlc/README.md) の「作業を始める前に考えておくこと」を参考に、**自分で決める**
   - 決められないことは「未確定。依頼者に確認する」と答えてかまいません
4. 答え終わると、AI-DLC が **答えの要約** を見せて確認してきます。合っていれば確定する
5. 文書（`intent-statement.md` など）ができ、AI のレビュー役が文書をチェックした結果が表で出ます
6. **承認ゲート** が出ます

   ```
   1. Approve          … 承認して次の段階（Requirements Analysis：要件定義）へ進む
   2. Request Changes  … 修正の内容を伝えて、直してもらう
   3. Other            … 別の希望を書く
   ```

7. 承認する前に、左のファイル一覧で `aidlc/…/ideation/intent-capture/` を開き、**できた文書を自分の目で読む**
8. 直してほしいところがあれば `2` を選んで理由を書く（例：`2 成功の基準に「定例の準備時間が10分以内になる」を足してください`）。よければ `1`

### 8. 段階2：要件分析（Requirements Analysis）（25分）

1. 同じように質問が来ます。今回は **ファイルに直接答える方法に切り替えて** みましょう
   - 答え方を聞かれた場合は **`2`（I'll edit the file）** を選ぶ
   - 「前回と同じ Guide me の方法で答えてもらいます」と、答え方を聞かれずに質問が始まることもあります。その場合はチャットで次を送る

   ```
   今回はファイルに直接答えます
   ```

   - チャットのまま答えることもできます。質問ごとの選択肢の番号をまとめて送れます（例：`Q1: 2、Q2: 3、Q3: 2`）

2. AI-DLC が示した質問ファイル（`requirements-analysis-questions.md` など）を開く
3. 各質問の下にある **`[Answer]:`** の後ろに答えを書く

   ```markdown
   ### Q3. 「進行中」に含めるステータスはどれですか？
   [Answer]: 見積中・開発中・テスト中。受付は含めない
   ```

4. **`Ctrl` + `S`** で保存し、チャットで次を送る

   ```
   答えを書き終えました
   ```

5. 答え終わると、文書を作る前に **答えの要約** が出て確認されます。合っていれば **`1`（Looks correct）**、直したいときは `2`（Request changes）

   ```
   1. Looks correct    … この回答をもとに文書を作る
   2. Request changes  … 作る前に回答を直す
   3. Other            … 別の希望を書く
   ```

6. 要件の文書ができたら、**受け入れ条件が「人が画面を見て確かめられる形」になっているか** を確認してから承認する
   - 例：「ステータスごとの件数が表示される」より「`変更依頼一覧_サンプル.csv` を読み込むと、開発中が5件と表示される」の方が確かめやすい

### 9. 段階3：コード生成（Code Generation）（20分）

1. poc では、作る計画（コード生成の計画）ができると、**確認を待たずに作り始めます**。「計画を先に見ますか？」という1行が出るので、見たいときは見せてもらう
2. `summary.html` などのファイルができていく様子を見守る
   - 途中で Kiro が「このコマンドを実行してよいか」と聞いてきたら、**何を実行しようとしているかを読んでから** 許可する。分からなければ許可せずに、「このコマンドは何をするものですか」とチャットで聞く
3. 承認ゲートが出たら、エクスプローラーで作業フォルダを開き、**`summary.html` をダブルクリック** してブラウザで開く
4. `変更依頼一覧_サンプル.csv` を読み込み、表示を確かめてから承認する

### 10. 段階4：ビルドとテスト（Build and Test）（15分）

1. 「検証に使うコマンド」を聞かれたら、**コマンドは使わない（手動で確認する）** と答える

   ```
   テストのコマンドはありません。technical-environment.md のとおり、手動確認チェックリストで確認します
   ```

2. AI-DLC が `summary.html`・**手動確認チェックリスト**（`手動確認チェックリスト.md`）・確認手順の文書を作ります。AI はブラウザを動かせないので、確認の目標は **Unverified（未確認）** のままです
3. 続けて **「Build Failure」** という画面が出ます。これは **失敗ではなく、人がまだ確かめていない** という意味です。AI-DLC は、人が確かめていないことを勝手に合格にしません

   ```
   1. Accept failure  … 未確認のまま記録して、承認に進む   ← 選ばない
   2. Abort           … ここで止める                       ← 選ばない
   3. Other           … チェックリストの結果を伝えるなど    ← これを使う
   ```

4. `手動確認チェックリスト.md` を開き、ブラウザで `summary.html` を開いて CSV を読み込み、**上から1つずつ自分で確かめる**（Edge だけで構いません）
5. チャットで **`3`** に続けて結果を送る

   ```
   3 Edge で全部 OK でした。Chrome は未確認です。
   ```

   - NG があれば、項目の番号とどうだったかを書く（例：`3 3-4 が NG：ベンダー別の表が見切れる`）。AI が直し方を提案してきます
6. 結果をもとに AI が確認の目標を **Met（確認済み）** に書き換え、承認ゲートが出ます。内容を確かめて **`1`（Approve）**
   - 確認の結果は `aidlc/spaces/default/intents/＜日付＞-＜名前＞/construction/build-and-test/test-results.md` にあります。確認の目標が **Met** になっていることを見てから承認する
   - フォルダが深くて探しにくいときは、`Ctrl` + `P` を押して `test-results` と入力すると直接開けます
7. 最後まで終わると、AI-DLC がワークフローの完了を知らせます

> **途中で時間切れになったら：** `/aidlc park` と送ると、その段階で一時停止します。次回、新しいチャットで aidlc エージェントを選び、`/aidlc --resume` と送ると続きから再開できます。今どこにいるかは `/aidlc --status` で確認できます。

### 11. 作業記録を見る・ふりかえり（20分）

1. 左のファイル一覧で `aidlc/spaces/default/intents/＜日付＞-＜名前＞/` を開き、次を見る

   | フォルダ・ファイル | 中身 |
   |---|---|
   | `ideation/intent-capture/` | 意図の把握の文書と、質問・答え |
   | `inception/requirements-analysis/` | 要件の文書と、質問・答え |
   | `construction/` | コード生成の計画、テストの記録 |
   | `aidlc-state.md` | どの段階まで終わったか |
   | `audit/` | 誰が・いつ・何を承認したかの記録。作業した PC ごとに `＜PC名＞-＜番号＞.md` というファイルができる（ファイル名に PC の名前が入るので、人に渡すときは注意） |

2. AI-DLC が **何でできているか** を見る。左のファイル一覧で `.kiro` を開くと、3つのフォルダに分かれています

   | フォルダ | 中身 | ハンズオンで近いもの |
   |---|---|---|
   | `.kiro/skills/`（約40個） | 手順書。`aidlc` が入口（チャットの `/aidlc`）で、段階ごとに1つずつある（`aidlc-intent-capture`、`aidlc-code-generation`、`aidlc-build-and-test` など）。`aidlc-mvp`・`aidlc-feature`・`aidlc-bugfix` などは型ごとの進め方 | STEP 9 のスキル |
   | `.kiro/agents/`（15個ほど） | 担当役。`aidlc-aidlc.md` がエージェント一覧の「aidlc」（進行役）で、ほかに設計・開発・品質・レビューなどの担当がいる | （新しく出てくるもの） |
   | `.kiro/hooks/` | 見張り役。人の返答を記録する（`record-human-turn`）、承認なしで先に進ませない（`plan-approval-guard`・`state-transition-guard`）、作業記録を残す（`write-audit-log`）など | STEP 5 のフック |

   - AI-DLC は、STEP 5・9 で使った **スキルとフックに、担当役のエージェントを組み合わせたもの** です。今回の `poc` で使ったのは、40個のスキルのうち段階1〜4の数個だけです
   - 中身は書き換えないでください（`aidlc doctor` の Framework integrity（部品がそろっているかの点検）で異常と判定されることがあります）

3. 次の問いの答えをメモに書き出す（仲間と行う場合は話し合う）
   - Spec（STEP 6〜7）と比べて、**人がやったこと** は何が違ったか
   - 「質問に答える」と「文書を直してもらう」では、どちらが楽だったか
   - 自分の仕事なら、どんな案件で使えそうか（例：関係部署が多い案件、監査で経緯を説明する必要がある案件）
4. 後片付け
   1. 最後の承認まで終わっていない場合は、チャットで `/aidlc park` と送って一時停止しておく（次回 `/aidlc --resume` で再開できます）
   2. チャット欄の下のモデル選択を **Auto に戻す**（Opus のままだと、ほかの作業でもクレジットを多く使います）
   3. エージェントは、新しいチャットを開けば Default に戻るので、操作は要りません
   4. Kiro で **「File」→「Close Folder」** を押して作業フォルダを閉じる
   5. 作業フォルダ（`C:\Users\＜自分＞\kiro\aidlc-taro`）は **そのまま残してかまいません**。リポジトリにはプッシュしないので、自分の PC の中にだけあります
      - 消したい場合は、Kiro を閉じてから PowerShell で次を実行する。**作業記録（`aidlc/`）も一緒に消え、元に戻せません**

        ```powershell
        Remove-Item -Recurse -Force $HOME\kiro\aidlc-taro
        ```

   6. AI-DLC 本体（`aidlc` コマンド）は、そのまま入れておいてかまいません

---

## よく出る英語

| 表示 | 意味 |
|---|---|
| Guide me / I'll edit the file / Chat | 1問ずつ聞いて／ファイルに書く／自由に話す |
| Approve / Request Changes | 承認する／差し戻す |
| Looks correct | （要約の確認で）合っている |
| Review outcome: Concerns remain for your decision | レビューで指摘が残っている。承認するかどうかは人が判断する |
| Severity: Minor / Major | 指摘の重さ：軽い／重い |
| Build Failure / Accept failure / Abort | （段階4）まだ人が確かめていない／未確認のまま進む／止める |
| Unverified / Met / Not Met | 未確認／確認済み（合格）／不合格 |
| Progress: 2 of 5 stages done. Next: … | 5段階中2段階が終わった。次は … |
| Skipped | この型では行わない段階なので飛ばした |

---

## できたかチェック

- [ ] AI-DLC をインストールし、`aidlc config` の最後の表で `0 of 8 sections need you`（すべて `[ok]`）を確認した
- [ ] Kiro でフォルダを信頼し、aidlc エージェントを選んで `/aidlc --doctor` を実行した
- [ ] `/aidlc poc` でワークフローを始め、質問に「Guide me」と「ファイルに書く」の両方で答えた
- [ ] 承認ゲートで、文書を読んでから Approve または Request Changes を選んだ
- [ ] `summary.html` をブラウザで開き、サンプルの CSV で動作を確かめた
- [ ] `aidlc/` の作業記録を開き、何が残っているかを説明できる
- [ ] モデルを Auto に戻した

## 困ったとき

- `aidlc` が見つからない（`aidlc : 用語 'aidlc' は … 認識されません`）→ PowerShell を開き直す。Kiro のターミナルなら、**Kiro を完全に終了して起動し直す**（Reload Window では直りません）
- エージェントの選択に **aidlc が出ない** → フォルダを信頼していない（画面上部に Restricted Mode の帯）。手順4をやり直し、Reload Window する
- `/aidlc` を送るたびに **「コマンドを実行してよいか」と何度も聞かれる** → エージェントが Default のままです。チャット欄で aidlc を選び直す
- 承認の番号を入れても **先に進まない・「人の返答が確認できない」と言われる** → フォルダの信頼と Reload Window を確認し、`/aidlc --doctor` の結果の警告を Kiro のチャットに貼って原因を聞く
- AI-DLC が **長い英語で答えてくる** → 「日本語で答えてください」と送る
- **段階が想定より多い**（例：15段階と表示された）→ 型が `poc` になっていません。始める前なら選び直す。始めてしまったら `/aidlc park` で止め、手順1から別の名前の作業フォルダ（例：`aidlc-taro2`）を作ってやり直す
- クレジットが残り少なくなった → `/aidlc park` で止め、翌月や追加のクレジットで `/aidlc --resume` から再開する
- そのほか → [困ったとき集](troubleshooting.md) の「STEP 10（AI-DLC）」
