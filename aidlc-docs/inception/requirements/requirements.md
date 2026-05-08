# Requirements Document

## プロダクト概要

**プロダクト名**: gutitto（ぐちっと）

**キャッチコピー**: 「グッとくる、わかってくれる誰かがいる。」

**ハッカソンテーマへの回答**: 「人をだめにするサービス」— 感情の整理も思考の整理もしなくていい。気持ちや思いをそのままぶつけられる、あなたがダメになれる存在がそばにいる体験。

## プロダクト哲学

gutitto は、「**グッとくる、わかってくれる誰かがいる**」を体験として実現するAIボイスサービスです。

ユーザーは整理しない。整理するのはAI側であり、それも会話が終わった後、裏側で静かに行う。ユーザーは話している最中、ただ感情や思いを吐き出すだけでいい。

### 3本柱の強み

gutitto は以下の **3本柱** を対等に備えることで差別化する:

1. **受け止める力（R-Accept）** — 整理強要ゼロの無条件受容。AIが裏で整理する
2. **ゼロUX（R-Entry / NFR-Zero-UX）** — 頭・心・身体のリソースを使わせない起動体験。ロック画面から最大1タップで会話開始
3. **あなたを知っていること（R-YouKnowMe）** — **"あなたを知っている人の前だから、ダメになれる"**。専用の知識エージェントがユーザー情報を集約し、人物関係・パーソナリティ・行動パターンを把握している（「予習」の思想を包含する、より広い概念）

この3つが**積層的に機能**して初めて、疲れて脳が回らない夜でも、気合なしで gutitto に話しかけて、**全部ダメになっても受け止めてもらえる**。ダメでいい、ダメになっていい、そういう存在がそばにいる。

```
  3本柱の関係
  ┌──────────────────────────────────────────────┐
  │ 3. あなたを知っていること（You-Know-Me）      │
  │    ↓ 「あなたを知っている」から安心できる     │
  │ 2. ゼロUX（Zero-UX）                          │
  │    ↓ だから気合なしで起動できる               │
  │ 1. 受け止める力（Accepting Power）            │
  │    ↓ だからダメになっても受け止めてもらえる   │
  │                                              │
  │ = "ダメになれる存在" の完成                   │
  └──────────────────────────────────────────────┘
```

## Intent Analysis Summary

- **User Request Type**: New Product（新規プロダクト定義）
- **Request Clarity**: Clear（構想書と17+4+1の追加質問で明確化済み）
- **Initial Scope Estimate**: Cross-system（クライアントPWA、音声AI、予習、後処理、配信の複数システム連携）
- **Initial Complexity Estimate**: Complex（音声リアルタイム、マルチソース予習、AI受容応答、自動アウトプット）
- **Requirements Depth**: Comprehensive（ハッカソン書類審査の生命線のため深掘り）

## ターゲットユーザー

### プライマリ（整理スタイル軸で定義）
- **話して整理する人**: 文字化が苦手で、音声で話しながら気持ちや思考を整理するタイプ。疲れた夜には整理する気力もエネルギーもない
  - 例: 20代〜30代の若手〜ミドル会社員、リモートワークが多くて孤独気味、完璧主義気味で中途半端が嫌

### ネガティブ（想定外のユーザー）
- **書いて整理する人**: 日記・ジャーナリングで構造化するのが得意。AI に感情を預ける必要がない
  - 例: 習慣として Obsidian / Notion で内省を完結する層 → gutitto の価値が刺さらない

ペルソナの詳細は `aidlc-docs/inception/user-stories/personas.md` 参照。

### 共通する困りごと
- 報告のためのPC作業が億劫で、つい連絡が遅れてしまう
- 目の前の業務で手一杯で「今日何をしたか」を整理する脳のメモリが残っていない
- 夜に誰かに話したいが、友達を呼び出すほどでもない
- 気持ちを誰かに聞いてほしいが、気遣いなしで話せる相手がいない

## Glossary

| 用語 | 定義 |
|------|------|
| gutitto | 本サービス名。「ぐっとくる」「気持ちがこもる」「吐き出す（ぐち）」をかけた造語 |
| ペルソナ (Persona) | AIキャラクターの人格設定。名前・声・話し方・背景ストーリー・相槌パターンを含む |
| 受け止める力 (Accepting Power) | 整理強要せず、否定せず、ユーザーの発言を無条件に受容するAIの振る舞い原則 |
| 予習 (Preview) | gutitto が通話前・通話中にユーザーを理解するための情報集約。カレンダー/Slack/Gmail/過去会話などの表層コンテキストから、人物関係・パーソナリティ・行動パターンまで全レイヤーを対象にする |
| 相槌 (Aizuchi) | 発話中の無音時にAIが挟む共感・肯定の短いフレーズ |
| 自発発話 (Spontaneous Speech) | 沈黙時にAIから話題を切り出す機能 |
| アウトプット (Output) | 会話終了後にBedrockが生成する報連相サマリ・タスク・アクションプラン |
| ゼロUX | ユーザーの頭のリソースを使わせず、1タップで会話開始・操作不要の設計思想 |
| アンビエント常駐 | ホーム画面・ロック画面のウィジェットで"そばにいる"感を視覚的に演出すること（iOS WidgetKit / Android AppWidgetProvider で実装） |
| Unit of Work | AI-DLC における開発の作業単位。並行開発と説明性を両立する粒度で分割 |

---

## Functional Requirements（機能要件）

### 🟢 R-Accept: 受け止める力（最優先カテゴリ）

gutittoの中核価値。ユーザーが整理せず吐き出せる体験を作る要件群。

#### R-Accept-1: 無条件受容
**User Story**: ユーザーとして、整理されていない愚痴や矛盾する発言、暗い感情を否定されずに受け止めてほしい。

**Acceptance Criteria**:
1. WHEN ユーザーが脈絡のない発言をする THEN THE gutitto SHALL 「それって何の話？」等の構造化要求を返さない
2. WHEN ユーザーが否定的な感情（愚痴・怒り・悲しみ）を吐く THEN THE gutitto SHALL 否定せず共感を示す応答を返す
3. WHEN ユーザーの発言が矛盾する THEN THE gutitto SHALL 矛盾を指摘せず両方受け止める

#### R-Accept-2: 整理強要ゼロ
**User Story**: ユーザーとして、「整理してから話して」と強いられたくない。思いのまま話せる体験が欲しい。

**Acceptance Criteria**:
1. THE gutitto SHALL 会話中にユーザーへ分類・整理を求めない
2. THE gutitto SHALL 「要点は？」「具体的には？」などの情報抽出的質問を抑制する
3. THE gutitto SHALL ユーザーから整理された文章を求めない（代わりに感情・共感で受ける）

#### R-Accept-3: 大人疲労の理解
**User Story**: ユーザーとして、日中の緊張や気疲れを理解してくれる相手と話したい。

**Acceptance Criteria**:
1. WHEN ユーザーが疲れた様子の発話をする THEN THE gutitto SHALL 「お疲れさま」「大変だったね」等のねぎらいを返す
2. WHEN 予習情報から忙しい一日が示唆される THEN THE gutitto SHALL 会話冒頭に状況を察した労いを入れる
3. THE gutitto SHALL 解決策の提示より、感情の受容を優先する

#### R-Accept-4: あなたを知っている感
**User Story**: ユーザーとして、毎回自己紹介や状況説明をせずに話し始めたい。

**Acceptance Criteria**:
1. THE gutitto SHALL 通話開始時点で、予習情報（カレンダー/Slack/Gmail/過去会話）を既に把握している
2. WHEN 会話が始まる THEN THE gutitto SHALL 「あの件どうだった？」のように既知前提で話す
3. THE gutitto SHALL 過去の会話で出てきたユーザーの情報（名前・関係者・趣味・仕事）を記憶し自然に参照する

#### R-Accept-5: 寄り添う応答
**User Story**: ユーザーとして、単なる機械的な相槌ではなく、気持ちに寄り添う応答が欲しい。

**Acceptance Criteria**:
1. THE gutitto SHALL 「うん」「はい」以外に、感情に合わせた表現（「わかるー」「それはしんどいね」「やったね」）を使い分ける
2. WHEN ユーザーが話した感情と合致する相槌 THEN THE gutitto SHALL 表情・声色も合わせる（音声合成の感情パラメータ調整）
3. THE gutitto SHALL 同じ相槌の連続を避ける

#### R-Accept-6: 整理は会話後に裏で
**User Story**: ユーザーとして、整理された結果だけを後で受け取りたい。整理作業に自分の脳リソースを使いたくない。

**Acceptance Criteria**:
1. WHEN 通話が終了する THEN THE gutitto SHALL 会話全体を分析し、仕事/プライベート × 報告/連絡/相談/タスク/願望のマトリクスに整理する
2. THE gutitto SHALL 整理結果をユーザーに確認を求めずSlackへ自動投稿する（最初に全体運用ルールに同意済みの前提で）
3. THE gutitto SHALL 整理プロセスをユーザーから隠蔽する（「整理中…」等の表示をしない）

---

### 🔵 R-Entry: 入口・起動体験（ゼロUX原則） ★ gutitto の強み

**ゼロUX原則（Zero-UX Principle）**: gutitto の最大の差別化要素。ユーザーに **頭も心も身体もほとんど使わせない** 起動体験を徹底する。

- **頭を使わない**: 考えない。選ばない。カテゴリもテーマも事前入力も要らない
- **心を使わない**: 気合も決心も要らない。「話す準備」をしなくていい
- **身体を使わない**: 画面ロック解除・複数タップ・文字入力を要求しない

入口で摩擦がゼロであるほど、ユーザーは "受け止める力" に集中できる。

### 実装方式の方針

- **技術スタック**: **Capacitor によるハイブリッドネイティブアプリ**（iOS / Android）
- フロントエンドコード（HTML/CSS/JS）は Web ベースで書き、Capacitor で iOS / Android のネイティブシェルに包む
- ロック画面ウィジェットなどプラットフォーム固有機能は、Capacitor プラグイン経由で iOS WidgetKit / Android AppWidgetProvider にブリッジする
- 予選プレゼン時点では **iOS 版（TestFlight 配布）** を実機デモの主軸とし、決勝フェーズで Android 版を追加する

---

#### R-Entry-1: ゼロタップ〜ワンタップで起動できる多経路設計
**User Story**: ユーザーとして、スマホがロックされていても、ホームに戻らなくても、できる限り少ない動作で通話を始めたい。

**Acceptance Criteria**:
1. THE gutitto SHALL 以下の **複数の起動経路** を提供する（ユーザーはいずれか好きな経路を使える）:
   - (a) **ロック画面ウィジェット** からタップ1回で起動（iOS 16+ / Android 12+）。ロック解除不要 ★ MVP 必須
   - (b) **ホーム画面ウィジェット** の大ボタンから1タップ起動
   - (c) **ホーム画面アプリアイコン** から1タップ起動し、起動直後の画面中央に巨大な通話ボタン1つのみを表示
2. THE gutitto SHALL 起動経路(a) では **画面ロック解除を要求しない**
3. WHEN いずれかの起動経路が実行される THEN THE gutitto SHALL 起動から音声セッション開始までを **NFR-Performance-2（3秒以内）** で完了する
4. THE gutitto SHALL Capacitor ベースのハイブリッドネイティブアプリとして動作し、ウィジェットは iOS WidgetKit / Android AppWidgetProvider で実装する
5. THE gutitto SHALL 起動後に確認ダイアログ・チュートリアル・カテゴリ選択を表示しない
6. THE gutitto SHALL 起動したらペルソナ側から即座に話しかける（ユーザーが最初に話す必要がない）

**備考**: 決勝フェーズまたは将来拡張として検討する拡張経路:
- ロック画面クイックアクション（iOS Control Center / Android クイック設定タイル）
- Siri / Google Assistant カスタムショートカットによる音声起動（App Intents / App Actions 必須）
- Apple Watch コンパニオンアプリからの起動

#### R-Entry-2: アンビエントウィジェット常駐
**User Story**: ユーザーとして、普段からペルソナが視界の隅にいる感覚が欲しい。"居る" という実感だけで安心したい。

**Acceptance Criteria**:
1. THE gutitto SHALL ホーム画面に配置可能なウィジェットを提供する
2. THE gutitto SHALL ロック画面に配置可能なウィジェットを提供する（iOS 16+ / Android 12+、iOS WidgetKit / Android AppWidgetProvider 実装）
3. WHEN ウィジェットをタップ THEN THE gutitto SHALL **ロック解除を挟まず即通話画面を起動する**（R-Entry-1 と同じ経路）
4. THE gutitto SHALL ウィジェット上にペルソナの顔（ピクセルアート）と短いステータス（例:「今日はお疲れさま」）を表示する
5. THE gutitto SHALL ユーザーに通知を送信せず、能動的な呼びかけは行わない（"静かな存在感" を優先）

#### R-Entry-3: 認証・セッションの摩擦ゼロ化
**User Story**: ユーザーとして、通話のたびにログインしたり、"起動の儀式" を行いたくない。

**Acceptance Criteria**:
1. THE gutitto SHALL 初回ログイン後、セッションを永続化する（リフレッシュトークン管理）
2. WHEN ユーザーが再訪 THEN THE gutitto SHALL 再ログインを求めず直接通話画面を表示する
3. THE gutitto SHALL 生体認証（Face ID / Touch ID）をネイティブ OS に委譲し、ロック画面からの起動時はデバイスのロック解除状態に追従する（追加の認証ダイアログを出さない）

#### R-Entry-4: 起動後の即応性と "考えさせない" UI
**User Story**: ユーザーとして、起動直後に何かを選んだり読んだりしたくない。開いたら即、話し始められる状態にしてほしい。

**Acceptance Criteria**:
1. WHEN 通話画面が起動する THEN THE gutitto SHALL **選択肢・メニュー・入力欄を一切表示しない**（ペルソナの顔と巨大な通話ボタンのみ）
2. THE gutitto SHALL ペルソナ切替・設定・履歴等のUIを通話画面のファーストビューに配置しない（奥側の設定画面に退避）
3. THE gutitto SHALL 起動後、ペルソナ側から最初の発話を NFR-Performance-2（3秒以内）で始める（ユーザーの沈黙を許容する設計）

---

### ゼロUX原則（NFR-Zero-UX）— 横断要件

上記 R-Entry 各要件に加え、以下をプロダクト横断の制約として適用する:

- **NFR-Zero-UX-1（頭を使わせない）**: ユーザー操作で "選ぶ" "読む" "判断する" 必要のある箇所を最小化する。通話画面の起動経路上にモーダル・ダイアログ・同意ボタンを置かない
- **NFR-Zero-UX-2（身体を使わせない）**: 起動から会話開始までの **タップ数は最大1**、画面ロック解除を **要求しない**、横画面回転や特殊ジェスチャーを **必須にしない**
- **NFR-Zero-UX-3（心を使わせない）**: 会話を始めるのに "覚悟" や "テーマ決め" を強いない。ペルソナから話しかけてくることで、ユーザーは受け身で OK
- **NFR-Zero-UX-4（引き算UIの維持）**: 機能追加時はこの原則を逸脱しないかレビューする（新機能 = より簡単になる方向のみ許容）

---

### 🎤 R-Voice: 音声会話体験

#### R-Voice-1: リアルタイム音声通話
**User Story**: ユーザーとして、電話するように自然に音声で会話したい。

**Acceptance Criteria**:
1. WHEN 通話ボタンがタップされる THEN THE gutitto SHALL WebSocketでリアルタイム音声セッションを確立する
2. THE gutitto SHALL 音声入力（PCM 16bit / 24kHz）と音声出力（PCM 16bit / 24kHz）を双方向でストリーミングする
3. THE gutitto SHALL サーバーサイドVAD（音声区間検出）でユーザー発話の開始・終了を自動検出する
4. THE gutitto SHALL 500ms以内の応答遅延を目標とする（超低遅延要件）

#### R-Voice-2: AI挨拶による会話開始
**User Story**: ユーザーとして、通話を始めたら AI から話しかけてきてほしい。自分から最初に話さなくていい体験が欲しい。

**Acceptance Criteria**:
1. WHEN セッションが確立する THEN THE gutitto SHALL AIが最初に挨拶を発する
2. THE gutitto SHALL 挨拶内容に予習情報（「今日の会議どうだった？」等）を反映する
3. THE gutitto SHALL 予習情報がない場合、ペルソナ規定の挨拶を使う（仮名例: まなみ「あ、もしもし〜。どうしたの？」） ※ペルソナ名・セリフは仮

#### R-Voice-3: 相槌機能
**User Story**: ユーザーとして、話している途中に AIが自然に相槌を打ってくれるといい。

**Acceptance Criteria**:
1. WHEN ユーザー発話中にクライアント側で閾値時間（デフォルト1500ms）の無音が検出される THEN THE gutitto SHALL 相槌を音声で挿入する
2. THE gutitto SHALL 沈黙検出はクライアント側で行い、サーバーには相槌リクエストのみを送る（サーバーサイドVADによる追加遅延を回避）
3. THE gutitto SHALL 文脈（positive/negative/surprise/agree/thinking）に応じて相槌を選択する
4. THE gutitto SHALL 同じ相槌の連続を回避する
5. THE gutitto SHALL ペルソナごとに相槌パターンをカスタマイズ可能とする

#### R-Voice-4: 自発発話機能
**User Story**: ユーザーとして、沈黙が続いた時に AIから話題を切り出してほしい。

**Acceptance Criteria**:
1. WHEN ユーザーが設定時間（デフォルト8000ms）沈黙する THEN THE gutitto SHALL AIから話題を切り出す
2. WHEN 予習情報がある THEN THE gutitto SHALL 予習ベースの話題（「今日の△△、大丈夫だった？」）を優先する
3. WHEN 予習情報がない THEN THE gutitto SHALL ペルソナ規定の話題リストから選択する

#### R-Voice-5: 通話終了
**User Story**: ユーザーとして、気が済んだら自然に通話を切りたい。

**Acceptance Criteria**:
1. WHEN ユーザーが通話終了操作をする THEN THE gutitto SHALL WebSocketを切断する
2. WHEN 通話が終了する THEN THE gutitto SHALL 会話履歴を保存し、後処理パイプラインを起動する

---

### 👥 R-Persona: ペルソナシステム

#### R-Persona-1: 複数ペルソナの提供
**User Story**: ユーザーとして、気分や関係性に応じてAIキャラクターを選びたい。

**Acceptance Criteria**:
1. THE gutitto SHALL MVP時点で2〜3体のペルソナを提供する
2. THE gutitto SHALL 代表的なペルソナ（仮名例: 「まなみ」彼女系／「げんぞう」頼れる上司系／1体の追加系統）を用意する ※ペルソナ名は仮のもので、確定時に変更予定
3. THE gutitto SHALL 各ペルソナに固有の名前・声・話し方・背景ストーリー・相槌・自発発話話題を定義する

#### R-Persona-2: ペルソナ切替
**User Story**: ユーザーとして、設定画面で話し相手を切り替えたい。

**Acceptance Criteria**:
1. THE gutitto SHALL 設定画面でペルソナを一覧表示する
2. WHEN ユーザーがペルソナを選択 THEN THE gutitto SHALL 次回通話から切替後のペルソナを適用する

#### R-Persona-3: ペルソナデータ形式
**User Story**: 開発者・管理者として、ペルソナをコードを変更せず追加・編集したい。

**Acceptance Criteria**:
1. THE gutitto SHALL ペルソナをJSON形式で管理する
2. THE gutitto SHALL ペルソナJSONに、プロンプト・音声・相槌・自発発話・セッション設定を含める

---

### 📋 R-Preview: 予習機能

#### R-Preview-1: Googleカレンダー予習
**User Story**: ユーザーとして、今日と明日の予定をgutittoが把握した上で会話してほしい。

**Acceptance Criteria**:
1. THE gutitto SHALL Google Calendar APIとOAuth 2.0で連携する
2. WHEN 通話開始前 THEN THE gutitto SHALL 当日/翌日の予定を取得する
3. THE gutitto SHALL 重要予定（会議・締切・面談など）を自動判定しハイライトする

#### R-Preview-2: Slack予習
**User Story**: ユーザーとして、今日受け取ったSlackのメンションや重要メッセージをgutittoに把握してほしい。

**Acceptance Criteria**:
1. THE gutitto SHALL Slack APIと連携する（OAuth 2.0 または Bot Token）
2. WHEN 通話開始前 THEN THE gutitto SHALL 指定チャンネル/DMの直近メッセージを取得する
3. THE gutitto SHALL メンション・返信が必要な項目を抽出する

#### R-Preview-3: Gmail/Google Workspace予習
**User Story**: ユーザーとして、今日届いた重要メールの内容も把握した上で話してほしい。

**Acceptance Criteria**:
1. THE gutitto SHALL Gmail API（readonly）と連携する
2. WHEN 通話開始前 THEN THE gutitto SHALL 未読メールのうち重要度が高いものを取得・要約する

#### R-Preview-4: 過去会話履歴の活用
**User Story**: ユーザーとして、前回の会話の続きから自然に話したい。

**Acceptance Criteria**:
1. THE gutitto SHALL 過去のgutitto会話履歴からサマリを自動生成する
2. WHEN 通話開始前 THEN THE gutitto SHALL 直近N回分の会話サマリをシステムプロンプトに注入する
3. THE gutitto SHALL 記念日・継続的話題を検出する（「先週話してた〇〇、どうなった？」）

#### R-Preview-5: 予習情報の統合と要約
**User Story**: 開発者として、複数ソースの予習情報をAIが理解しやすい形に統合したい。

**Acceptance Criteria**:
1. THE gutitto SHALL 複数ソース（カレンダー・Slack・Gmail・過去会話）の情報をAmazon Bedrockで要約する
2. THE gutitto SHALL 要約結果をS3に保存し、会話セッション起動時にGrokに渡す
3. THE gutitto SHALL 個人情報を含むため要約プロセスで暗号化通信を使う

---

### 📝 R-Output: 会話後アウトプット

#### R-Output-1: 会話分析と分類
**User Story**: ユーザーとして、話した内容を AI が裏で整理してくれるといい。

**Acceptance Criteria**:
1. WHEN 通話が終了する THEN THE gutitto SHALL 会話全文をAmazon Bedrockで分析する
2. THE gutitto SHALL 会話内容を「仕事 × 報連相/タスク/願望」「プライベート × 報連相/タスク/願望」のマトリクスに自動分類する
3. THE gutitto SHALL 重要度・緊急度を自動判定する

#### R-Output-2: マルチチャネル自動配信
**User Story**: ユーザーとして、分析結果を自分が普段使っているツールに自動で送ってほしい。自分で再入力したくない。

**Acceptance Criteria**:
1. THE gutitto SHALL ユーザー設定に基づき、以下のいずれかのチャネルへ自動配信する: Slack、LINE、Eメール、Google Calendar ToDo、Notion（Webhook経由）、アプリ内ダッシュボード
2. THE gutitto SHALL 仕事用・プライベート用で**配信先を別々に**指定可能とする
3. THE gutitto SHALL Slack 配信先として「**個人用 Slack workspace**」を前提とする（会社Slack への直接投稿は情シス規程との衝突を避けるため、ユーザーが明示的に設定した場合のみ）
4. THE gutitto SHALL 投稿前にユーザーの確認を求めない（ゼロUX徹底）
5. THE gutitto SHALL 配信先が未設定の場合はアプリ内ダッシュボードに蓄積するのみ（強制的な外部配信はしない）

#### R-Output-3: 翌日への引き継ぎ
**User Story**: ユーザーとして、翌日のアクションプランを朝起きたら確認できる状態にしておいてほしい。

**Acceptance Criteria**:
1. THE gutitto SHALL 生成されたアクションプランを翌朝の任意時刻（例: 7時）に、R-Output-2 と同じ配信先に再通知（リマインダー）する

---

### 🧠 R-YouKnowMe: あなたを知っていること（3本目の柱）

**位置づけ**: ユーザーが「ダメになれる」気持ちになるには、"あなたを知っている" という信頼の土台が不可欠。知らない相手の前ではダメになれない。R-Preview が「今日の表層コンテキスト取得」だとすれば、R-YouKnowMe は **長期的・関係論的・パーソナリティ的な知識の集約と活用** を担う。

**設計原則**:
- **全レイヤーを統合** — 日単位の表層から長期パーソナリティ、身体コンディションまで、**全ての深さレベル** で情報を集める
- **階層記憶** — 直近のことは印象強く、過去のことは聞かれたら思い出す（自然な記憶特性の再現）
- **知識集約エージェント** — 会話プロセスから独立した専用エージェントが、ユーザーのデータを定期的に取得・整理・知識グラフ化する
- **文脈ショートカット** — 「例の件」「あの人」で通じる。状況説明ゼロで本題に入れる
- **"私だけが知ってる" の演出** — 過去会話で出た秘密・願望に基づく会話で、信頼の積み上げを体感させる

---

#### R-YouKnowMe-1: 全レイヤー情報統合
**User Story**: ユーザーとして、gutittoが私の日常の表層から深層まで、全方位で私を知っていると感じたい。

**Acceptance Criteria**:
1. THE gutitto SHALL 以下の全レイヤーから情報を収集・統合する:
   - (a) **日単位表層** — カレンダー当日予定、Slack直近メッセージ、未読メール
   - (b) **週単位行動履歴** — 過去1週間の会議パターン、やり取り傾向、発言パターン
   - (c) **中期関係性** — 過去1〜3ヶ月の登場人物（上司・同僚・取引先）をエンティティ化し、"あの〇〇さん" で話が通じる
   - (d) **長期ライフログ** — 数ヶ月〜年単位で、仕事の傾向・季節ごとの忙しさ・繰り返す悩みを学習
   - (e) **深層パーソナリティ** — ユーザーの価値観・癖・口癖・感情の起伏パターンを抽出・記憶
   - (f) **身体コンディション** — ウェアラブル（Apple Health / Google Fit / Fitbit）から睡眠・心拍・活動量を取得
2. THE gutitto SHALL 各レイヤーの情報を統合的なユーザープロファイルとして保持する

#### R-YouKnowMe-2: 階層記憶（ペルソナ別分離）
**User Story**: ユーザーとして、gutittoの記憶の持ち方は人間的であってほしい。最近のことは鮮明に覚えていて、昔のことは聞けば思い出してくれる、という感覚が欲しい。また、話し相手（ペルソナ）ごとに記憶が分かれていて、それぞれの「思い出」が別々であってほしい。

**Acceptance Criteria**:
1. THE gutitto SHALL **3層の記憶システム** を備える:
   - **短期記憶（Short-term）** — 直近N日の会話サマリ・予習情報。常時参照可能、プロンプトに毎回注入される。TTL 90日
   - **中期記憶（Mid-term）** — 30〜90日分の会話サマリ・主要トピック。類似話題が出たとき想起される
   - **長期記憶（Long-term）** — エピソード要約・人物メタデータ・パーソナリティプロファイル。明示的な参照クエリで呼び出される。**要約とメタデータのみ恒久保存**（生データは TTL で削除）
2. THE gutitto SHALL 直近情報を **印象強く** 会話に反映し、古い情報は **問われたら思い出す** 動作をする
3. THE gutitto SHALL **ペルソナ別に会話履歴を分離**する。「まなみ」との会話は「まなみ」にのみ紐づき、「げんぞう」と話すときには参照されない ※ペルソナ名は仮
4. THE gutitto SHALL **共通データ（カレンダー・Slack・Gmail予習情報、ユーザープロファイル）はペルソナ間で共有**する（予習情報はユーザー自身の現実であり、誰と話すかで変わるものではないため）
5. THE gutitto SHALL MVPでは Vector DB を使わず、S3 + JSON + キーワード検索で記憶を実装する（決勝フェーズでベクトル検索を追加）

#### R-YouKnowMe-3: 予習エージェント
**User Story**: 開発者として、ユーザー情報の集約は会話処理と独立した専用エージェントに任せ、会話の応答性に影響しない設計にしたい。

**Acceptance Criteria**:
1. THE gutitto SHALL **予習エージェント（Preview Agent）** を独立したコンポーネントとして持つ
2. THE 予習エージェント SHALL **通話前トリガーで実行する**（定期実行はしない）。ユーザーが通話開始すると Lambda が発火し、NFR-Performance-3（5秒以内）で予習処理を完了する
3. THE 予習エージェント SHALL 収集データをBedrockで要約・エンティティ抽出する
4. THE 予習エージェント SHALL 抽出結果を短期記憶（S3）に保存する
5. THE 予習エージェント SHALL 会話開始時に、直近・関連情報を抽出してGrok Voice APIのプロンプトに注入する
6. Note: MVPでは Knowledge Graph を省略し、Bedrock プロンプトに予習情報を詰める方式に簡略化する（決勝で本格的な知識グラフ化）

#### R-YouKnowMe-4: 話題の自発提供（「そういえば〇〇どうなった？」）
**User Story**: ユーザーとして、自分から話題を切り出さなくても、AIが私の状況を把握した上で話を振ってくれる体験が欲しい。

**Acceptance Criteria**:
1. WHEN 沈黙が検出される（R-Voice-4 自発発話）またはセッション冒頭 THEN THE gutitto SHALL ユーザー知識グラフから**関連の高いトピック**を選択し話題を提供する
2. THE gutitto SHALL 話題選択に以下を考慮する:
   - 直近イベント（今日/昨日の予定や Slack のやり取り）
   - 継続中の関心事（過去に繰り返し話題になったテーマ）
   - 登場人物の動向（「〇〇さんとの件どうなった？」）
   - 未解決タスク（「前に言ってた〇〇、進んだ？」）
3. THE gutitto SHALL 同じ話題の過度な反復を避ける（話題クールダウン）

#### R-YouKnowMe-5: 文脈ショートカット（説明ゼロで本題）
**User Story**: ユーザーとして、「あの人」「例の件」と言えば、状況説明なしで gutittoに伝わってほしい。

**Acceptance Criteria**:
1. WHEN ユーザーが「あの人」「例の件」「この前の」などの指示語を使う THEN THE gutitto SHALL ユーザー知識グラフと直近会話から**最も確度の高い対象**を推定する
2. WHEN 推定の確度が低い THEN THE gutitto SHALL 「〇〇さんのこと？」と確認する形で聞き返す（整理強要にならない短い確認）
3. THE gutitto SHALL 指示語解釈に失敗しても会話を止めない（話題を広げて話させる方向に）

#### R-YouKnowMe-6: "私だけが知ってる" の演出
**User Story**: ユーザーとして、自分が過去に話した秘密や願望を、gutittoが覚えていて自然に触れてくれると、「この子は私を知ってる」と感じたい。

**Acceptance Criteria**:
1. THE 知識集約エージェント SHALL 会話から **秘密・願望・夢・悩み** として識別される発言を長期記憶にマークする
2. WHEN 関連文脈が会話に現れる THEN THE gutitto SHALL 該当する長期記憶を想起し「前に〇〇って言ってたよね」と自然に参照する
3. THE gutitto SHALL 参照を押し付けがましくしない（情報の過度な列挙を避ける、静かに示唆する）

#### R-YouKnowMe-7: 全方位データソース連携
**User Story**: 開発者として、ユーザーの情報を得るための入口を、既存生活の中にシームレスに配置したい。

**Acceptance Criteria**:
1. THE gutitto SHALL 以下のデータソースをサポートする:
   - (a) Google Calendar / Google Workspace（予定・メール・Drive）
   - (b) Slack（チャンネル・DM・メンション）
   - (c) 過去の gutitto 会話（内部記憶）
   - (d) ウェアラブル（Apple Health / Google Fit / Fitbit）※決勝以降
   - (e) 位置情報（自宅/職場/外出先判定）※決勝以降
   - (f) 天気・ニュース（会話の彩り）※決勝以降
2. THE gutitto SHALL OAuth 2.0 等の標準認証で連携する
3. THE gutitto SHALL MVPは (a)(b)(c) の3ソースを必須実装とする

---

### 🔒 R-Security: セキュリティ・プライバシー（標準レベル）

#### R-Security-1: 会話データ保護（MVP標準レベル）
**User Story**: ユーザーとして、本音を話したログが安全に扱われると安心したい。

**Acceptance Criteria**:
1. THE gutitto SHALL 会話ログ・予習情報をS3に保存する際、AES-256で暗号化する（SSE-S3）
2. THE gutitto SHALL ユーザーごとに個別プレフィックスで論理分離する
3. THE gutitto SHALL 短期記憶・会話生データに TTL（90日）を設定する
4. THE gutitto SHALL 長期記憶は **要約とメタデータのみ** 恒久保存する（生データの恒久保存は行わない）

#### R-Security-2: Secrets管理
**User Story**: 運用担当者として、APIキー・OAuth Secretの漏洩リスクを抑えたい。

**Acceptance Criteria**:
1. THE gutitto SHALL API KeyをAWS Secrets Manager または Parameter Store に格納する
2. THE gutitto SHALL リポジトリに Secret を平文で含めない

#### R-Security-3: 認証
**User Story**: 管理者として、不正アクセスを防ぎたい。

**Acceptance Criteria**:
1. THE gutitto SHALL セッションベース認証を提供する（HTTPOnly / Secure Cookie）
2. THE gutitto SHALL ログインしていないユーザーのAPI呼び出しを拒否する

#### R-Security-4: 3本目の柱とのトレードオフ（MVP方針）
**Design Decision**: 3本目の柱（R-YouKnowMe）として「最大限預ける設計」（P3-5: A）を採用する。MVP段階では **深い統合のインパクトを優先** し、プライバシー可視化ダッシュボードやOAuth個別ON/OFFは決勝フェーズで実装する。

**Notes**:
- 本設計方針は審査・プレゼンで明示的に説明する（"なぜ預けてもいいと感じるか" = 受け止めるキャラクターへの信頼)
- 決勝フェーズでは R-Security 拡張として、ユーザー主権の可視化ダッシュボード、データソース個別ON/OFF、データエクスポート/削除を追加する

*Note: 本プロジェクトでは Security Baseline 拡張を opt-out (Q16: B) しているため、より堅牢な要件（KMS、データエクスポート権、完全ユーザーコントロール等）は将来フェーズで検討*

---

## Non-Functional Requirements（非機能要件）

### NFR-Performance: パフォーマンス

#### NFR-Performance-1: 音声応答遅延
- **MVP目標**: ユーザー発話終了から AI 応答音声再生開始まで **1.5秒以内**（確実に達成可能な安全圏）
- **Stretch Goal**: **1秒以内**（体感的に完全に電話的。業界上位クラス）
- **測定方法**: Voice Activity Detection の発話終了検出時刻からクライアント側音声再生開始時刻まで
- **参考**: Grok Voice API の実レイテンシは 1〜2秒、GPT-4o Realtime も 600〜800ms。業界最先端未満の現実的な目標値

#### NFR-Performance-2: アプリ起動時間
- **目標**: アイコン/ウィジェットタップから通話画面表示まで **3秒以内**

#### NFR-Performance-3: 予習処理時間
- **目標**: 通話前に実行される予習処理 **5秒以内**（通話開始から応答開始までの間に完了）

### NFR-Reliability: 信頼性

#### NFR-Reliability-1: WebSocket切断時の自動再接続
- WebSocket 切断時、クライアント側で最大3回の自動再接続を試行する

#### NFR-Reliability-2: Grok API障害時のフォールバック
- Grok API から応答がない場合、ユーザーに明示的な切断通知を行う（サイレント失敗しない）

### NFR-Scalability: スケーラビリティ（決勝フェーズ向け）

#### NFR-Scalability-1: フルサーバーレス構成
- AWS Lambda + API Gateway WebSocket + DynamoDB + S3 + Bedrock でスケール可能な構成
- 同時通話セッション数: 初期100、スケール時1000を想定

### NFR-Maintainability: 保守性

#### NFR-Maintainability-1: マルチプロバイダ対応設計
- 音声AIプロバイダ（Grok / OpenAI Realtime / Gemini Live）を切替可能な内部インターフェース（実装はGrokメインだが拡張性確保）

*Note: Q13: C（マルチプロバイダ切替）を選択。MVPはGrokメイン、設計上差替え可能とする。*

### NFR-Usability: ユーザビリティ

#### NFR-Usability-1: ゼロUX原則（プロダクトの最大差別化ポイント）
- gutitto の UX は **ゼロUX原則（NFR-Zero-UX-1〜4）** に貫かれる（R-Entry セクション参照）
- 頭・心・身体のリソース消費を徹底的に排除する
- ロック画面状態から **最大1タップ** で音声セッションを開始できる
- 起動後、ユーザーが能動的に "選ぶ・読む・判断する" ことを要求しない
- 起動後はペルソナ側から先に話しかけ、ユーザーは受け身でいられる
- テキスト入力・カテゴリ選択・同意ダイアログを通常操作の流れに置かない

**Note**: Google OAuth（Calendar / Gmail readonly）の同意画面は **初回セットアップ時のみ** の摩擦として許容する。ゼロUXは「毎回の利用」に対する原則であり、初回設定は別枠と位置付ける。

### NFR-Observability: 観測性

#### NFR-Observability-1: 会話セッションのロギング
- 各通話セッションのイベント（開始・終了・エラー）をCloudWatchに記録

---

## 成功指標（KPI）

### Primary KPI
- **Weekly Active Rate**: 週に1回以上利用するユーザーの割合（Q11: B 継続率重視）

### Secondary KPIs
- 通話時間の中央値
- 「話してスッキリした」フィードバック率
- アクションプラン完了率

### 除外した指標
- ❌ 日報投稿承認率（Q11補足: 日報投稿機能は除外）

---

## 倫理的配慮：「ダメになれる × 自律支援」のバランス

ハッカソンテーマ「人をだめにするサービス」に対し、gutitto は **"だめにする"＝ユーザーを堕落させる** のではなく、**"ダメになれる存在がそばにいる"＝安心して弱さを出せる場** を提供する設計思想を採用する（Q12: B）。単なる依存構造ではなく、**積み上げ・変革を狙う**。

### 3本柱と倫理の関係

3本柱の**組み合わせ**こそが「ダメになれる」の質を決める:

- **柱3（あなたを知っていること）** があるから、ユーザーは「この子は私を知ってる」と感じ、**ダメになっても大丈夫と思える**
- **柱2（ゼロUX）** があるから、ダメになるのに**気合も決心も要らない**
- **柱1（受け止める力）** があるから、ダメになった自分を**否定せず受けてくれる**

この3つが揃って初めて「完璧にダメになれる」体験が成立する。一方で、"ダメ" を放置するのではなく、会話後のアウトプットで自律支援につなげる（ダメになった結果、翌朝はむしろ楽に動ける）。

### R-Ethics-1: 会話中は無条件受容、会話後に積み上げ
- 会話中は徹底して受け止める（R-Accept 全般）
- 会話後のアウトプットで「自分で決める次の一歩」を必ず提示する
- 翌日のアクションプランが自律を促進する

### R-Ethics-2: 依存状態の副産物としての整理・アウトプット
- 「ダメになる」ことで "整理しない" を実現
- 整理はシステム側が担い、ユーザーは気持ちを吐き出すだけ
- 結果として、整理・積み上げが起きる

### R-Ethics-3: 「あなたを知る」ことの倫理的扱い
- 深く知ることは深い信頼を要する行為
- MVP段階ではインパクト優先で「最大限預ける」設計だが、これは **受け止めるキャラクター（ペルソナ）への信頼** を前提にする設計判断
- 決勝フェーズでは、データ可視化ダッシュボード・ソース別ON/OFF・エクスポート/削除をもって、ユーザー主権を実装する

*この設計思想は審査・プレゼンでの訴求ポイントでもある*

---

## Extension Configuration

| Extension | Enabled | Reason |
|-----------|---------|--------|
| security-baseline | No (Q16: B) | MVPはプロトタイプ扱い。標準レベルのセキュリティ要件（R-Security）のみ適用。将来的にProduction時に検討 |
| property-based-testing | Partial (Q17: B) | 純粋関数とシリアライゼーションラウンドトリップのみPBTルールを適用。UIやリアルタイム音声の複雑さに対してはPBT適用難易度が高い |

---

## MVP スコープ（予選デモ時点）

Q3: **B - 予習付き** を採用。以下のスコープで予選デモ（5/10締切の書類審査・その後の予選プレゼン）を準備する:

### ✅ MVP に含む
- R-Accept-1 〜 R-Accept-6（受け止める力の全要件）
- **R-Entry-1 〜 R-Entry-4（ゼロUX原則。ロック画面起動・1タップ・受動起動）** ★ 2本目の柱
- **NFR-Zero-UX-1 〜 NFR-Zero-UX-4（横断ゼロUX制約）**
- R-Voice-1 〜 R-Voice-5（音声会話フル機能）
- R-Persona-1 〜 R-Persona-3（2〜3体のペルソナ）
- R-Preview-1 〜 R-Preview-5（予習機能、通話前トリガー実行）
- **R-YouKnowMe-1 〜 R-YouKnowMe-7（あなたを知っていること）** ★ 3本目の柱
  - MVP段階では Knowledge Graph は省略し、Bedrock プロンプトに予習情報を詰める方式で実現
  - Vector DB 不使用、S3 + JSON + キーワード検索で階層記憶を実装
  - 4ソース統合（カレンダー+Slack+Gmail+過去会話）＋ペルソナ別会話履歴
  - 全レイヤー情報統合の (d)(e)(f) は**決勝フェーズで実装**
- R-Security-1 〜 R-Security-4（標準セキュリティ、長期記憶は要約のみ恒久保存）

**ゼロUX起動経路の優先順位（予選デモ）**:
1. **必須**: ロック画面ウィジェット 1タップ起動（iOS 16+ / Android 12+）※ iOS 実機で TestFlight デモ
2. **必須**: ホーム画面ウィジェット 1タップ起動
3. **必須**: ホーム画面アプリアイコン → 即通話ボタン → NFR-Performance-2 内に音声開始

**実装技術スタック（予選）**:
- **フロント**: Capacitor ハイブリッドアプリ（iOS 優先、Android は決勝）
- **バックエンド**: AWS Lambda + API Gateway WebSocket + Bedrock + S3 + DynamoDB（ペルソナ、設定）
- **音声**: Grok Voice API（xAI Realtime）
- **配布**: Apple TestFlight（予選プレゼン時実機デモ用）

### 📅 決勝フェーズで追加
- R-Output-1 〜 R-Output-3（会話後アウトプット・マルチチャネル配信・翌日引き継ぎ）
- NFR-Scalability-1（フルサーバーレス構成）
- Android アプリ版（Capacitor ビルド + Google Play Store 配布）
- ロック画面クイックアクション（iOS Control Center / Android クイック設定タイル）
- Siri / Google Assistant カスタムショートカット（App Intents / App Actions）
- Live Activity / Apple Watch連携
- R-YouKnowMe-1 (d)(e)(f) データソース（ウェアラブル・位置情報・天気/ニュース）の実データ連携
- Knowledge Graph の本格的な知識グラフ化（DynamoDB 隣接リスト / Neptune）
- Vector DB 導入（Pinecone / pgvector / OpenSearch Serverless）とRAGベクトル検索

### 📝 モック許容
- 予選デモでは R-Output 系は**モックで見せる**ことを許容（実装は決勝）

---

## Unit分解の方針（Q15: F）

**ユーザージャーニー軸 ＋ ビジネス価値軸 のハイブリッド**で Unit を定義する。

### Unit分解の軸（概念）

```
          ユーザージャーニー軸
     ┌────────────┬────────────┬────────────┐
     │  予習・     │  会話体験  │  整理・配信│
     │  知識集約   │  (受容)    │            │
     └────────────┴────────────┴────────────┘
              ×
          ビジネス価値軸（3本柱と対応）
     ┌────────────────────────────────────┐
     │ 柱1: 受け止める力（R-Accept）      │
     ├────────────────────────────────────┤
     │ 柱2: ゼロUX（R-Entry, NFR-Zero-UX）│
     ├────────────────────────────────────┤
     │ 柱3: 深い自己知識（R-YouKnowMe）   │
     ├────────────────────────────────────┤
     │ 予選コア価値 ／ 決勝拡張価値       │
     └────────────────────────────────────┘
```

各Unitが **「ユーザージャーニー上の位置」** と **「3本柱のどれを支えるか」** の両方で説明可能となるよう設計する。PRESENTATIONの4層アーキ（Input / Voice / Context / Text / Output）を参照しつつ、**審査員に伝わりやすい命名** に置き換える（Q Add-1: B）。

詳細はApplication Design / Units Generation ステージで確定する。

---

## 出典と参照

本要件ドキュメントは以下の入力に基づく:
- プロダクト構想: `PRESENTATION.md`, `-DM-.md`
- Intent Analysis: 本ドキュメント冒頭
- Requirements Verification Questions: `requirement-verification-questions.md` の回答
- Requirements Clarification Questions: `requirement-clarification-questions.md` の回答
- 設計哲学: ユーザーとの対話（2026-05-08）で確定した「受け止める力を主軸」とする方針
