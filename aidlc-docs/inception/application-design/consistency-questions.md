# Consistency & Feasibility Questions

`consistency-feasibility-review.md` で発見した矛盾・実現可能性リスクを解決するための質問ファイル。

各質問の `[Answer]:` タグに回答してください。

---

## Question Q1: PWA vs ネイティブ（★最重要）

**問題**: R-Entry-1 で挙げた起動経路のうち、以下は **PWA では技術的に実現不可能**:
- (a) ロック画面ウィジェット（iOS 16+ は WidgetKit + Swift 必須）
- (b) ロック画面クイックアクション（ネイティブ）
- (c) ホーム画面ウィジェット（Android は AppWidgetProvider 必須）
- (e) Siri / Google Assistant ショートカット（App Intents / App Actions 必須）

PWA で可能なのは:
- (d) ホーム画面アプリアイコン
- Web Push 通知
- iOS 18+ の Web Push（制限あり）
- 生体認証は WebAuthn で類似可能

### 選択肢

A) **PWA一本で現実的スコープに縮退** — R-Entry-1 は (d) のみを必須とし、(a)(b)(c)(e) は削除。MVPは「ホーム画面アイコン1タップ起動」に集約。ウィジェット・Siriは書類上「将来拡張」として記載
B) **ハイブリッド（PWA＋軽量ネイティブシェル）** — Capacitor / Tauri Mobile で PWA をネイティブシェルに包み、iOS/Android ウィジェットと Siri ショートカットを実装。MVP実装負荷は中〜高
C) **完全ネイティブ（iOS Swift + Android Kotlin）** — ウィジェット完全対応だが、ハッカソン期間で開発不可。予選はモック画面でのみデモ
D) **MVPはPWA (d) 一本＋モック動画** — 予選デモは PWA だけで動かし、ウィジェット／Siri 起動は **プレゼン動画** で見せる。決勝通過時にネイティブ化
X) Other (please describe after [Answer]: tag below)

[Answer]: 
(a)を実現してみたいが、どれくらい工数負荷がかかりそうか
---

## Question Q2: 500ms 応答遅延目標

**問題**: 500ms は Grok Voice API の実レイテンシ（1〜2秒）を考えると達成困難。業界最先端（GPT-4o Realtime）でも 600〜800ms クラス。

### 選択肢

A) **800ms に緩和** — 業界最先端クラスを目指す。実測による調整余地あり
B) **1秒に緩和** — 体感的にまだ「電話的」。安全圏
C) **1.5秒に緩和** — 確実に達成可能、ベストエフォート
D) **「500ms を目指すが、ベストエフォート」として目標値と現実値を併記** — NFR として両方明記
E) **Stretch Goal として 500ms を残し、MVP目標は 1秒** — 二段階設定
X) Other (please describe after [Answer]: tag below)

[Answer]: 
Cでよいが、Bは目指したい
---

## Question Q3: Siri / Google Assistant ショートカット

**問題**: PWA では Siri カスタムショートカット登録不可。URL起動ショートカットは可能だが「Hey Siri、まなみ」は使えない。

### 選択肢

A) **削除** — ゼロUX原則を守りつつも、Siriは決勝ネイティブ化時のみ対応として要件から削除
B) **URL起動ショートカットで代替** — ユーザーが「gutittoを開く」などの標準ショートカットを手動登録する形で "それっぽく" 実現（Hey Siri ウェイクは標準音声のみ）
C) **Q1でネイティブ採用の場合のみ実装** — Q1 の回答に従属
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question Q4: Knowledge Graph の MVP 必要性

**問題**: 現設計では Knowledge Graph を独立コンポーネントとして DynamoDB で実装する想定。ただし MVP の Must 5本（1.1/2.1/2.2/2.3/4.1）を動かす範囲では、**Bedrock プロンプトに予習情報を詰めるだけ** でほぼ成立する。

### 選択肢

A) **MVPでは Knowledge Graph を省略** — Preview Agent が Bedrock で要約した直近コンテキストを毎回プロンプトに詰める。「指示語解釈」「人物関係」は決勝追加
B) **簡易 KG をMVPに含める** — DynamoDB で人物エンティティのみ管理（関係性エッジは決勝）
C) **完全 KG を MVP に含める** — 予習 Unit 担当が頑張って実装する（決勝の差別化に強い）
D) **KG という呼び方を捨て、「構造化メモリ」に改名** — 過度な技術印象を避け、実装は簡易に
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question Q5: Vector DB の選定

**問題**: OpenSearch Serverless は最低 $350/月からスタート。予選デモのコストに不適。

### 選択肢

A) **Pinecone Starter（無料枠）** — 月100K ベクトル無料。予選に最適
B) **pgvector（RDS PostgreSQL）** — $15〜/月（db.t4g.micro）、運用経験ある人が多い
C) **ChromaDB（Lambdaローカル）** — サーバーレスだが耐久性なし、予選デモ向き
D) **MVPでは Vector DB 使わない** — 直近記憶のみ S3 + JSON で。ベクトル検索は決勝で追加
E) **OpenSearch Serverless で進める** — コスト承知で本格構成
X) Other (please describe after [Answer]: tag below)

[Answer]: 
D
---

## Question Q6: 予習エージェントの実行トリガー

**問題**: 定期実行と通話前トリガーの両対応は複雑度・コスト高。

### 選択肢

A) **通話前トリガーのみ** — ユーザーが通話開始 → Lambda が予習を走らせる → 500ms〜2秒以内に完了してセッション注入
B) **定期実行のみ（1時間ごと）** — 常に最新予習が準備されている状態。通話時は即座に使える
C) **ユーザー毎アクセス時にキャッシュ失効をトリガー** — 前回予習から一定時間経過していれば通話前に再予習
D) **通話前 + 直前30分以内のキャッシュ利用** — 通話前トリガーするが、直近キャッシュあれば再利用
X) Other (please describe after [Answer]: tag below)

[Answer]: 
MVPとしては一番何度が低いものを設定
---

## Question Q7: 長期記憶のスキーマ

**問題**: 恒久保存は長期的にコスト爆発。

### 選択肢

A) **長期記憶は "要約 + 人物メタデータ" のみ恒久保存** — 会話生データは90日TTLで削除、サマリとエンティティメタデータだけ残す
B) **全文恒久保存** — プライバシーリスク承知で、ユーザー体験優先
C) **ユーザー選択式** — 設定でデータ保持期間を選べる（デフォルトA）
D) **MVPは A、決勝で C に拡張**
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question Q8: Accepting Engine の応答品質保証方式

**問題**: 「validateResponse で検出・抑制」は事後チェックで、発話済みのため実時間で止められない。

### 選択肢

A) **事前プロンプトのみに統一** — Accepting Engine は「プロンプト構築」に特化。validateResponse は削除。ログ記録は Observability に寄せる
B) **事前プロンプト + ログ監視** — 発話は許容し、品質監視は KPI 改善のためのログ記録目的
C) **Gate方式** — LLM応答テキストを Grok が発話する前に別 LLM で検証（追加レイテンシ発生、実現困難）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question Q9: ペルソナ切替時のコンテキスト継承

**場所**: stories.md 2.6

**問題**: 「まなみ」との過去会話を「げんぞう」が参照するか。「前の彼氏の話を覚えている彼氏」になりかねない。

### 選択肢

A) **ペルソナ間でコンテキスト完全共有** — めぐみの情報は誰が話し相手でも同じ（AIだから自然）
B) **ペルソナ別に記憶分離** — 「まなみとの思い出」「げんぞうとの思い出」が別。より人間的、キャラクター性強い
C) **共通データ（カレンダー/Slack予習）は共有、会話履歴はペルソナ別**
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## Q10-13（軽微 / 自動修正）

以下は回答不要、AI判断で修正します（問題があれば指摘してください）:
- **Q10**: Requirements のターゲットユーザーを Personas の切り口に合わせて更新
- **Q11**: Slack 投稿先を「個人用 Slack workspace を前提」と明記
- **Q12**: Google OAuth の同意画面重さを初回セットアップ時のみの許容として注釈
- **Q13**: 相槌タイミングをクライアント側検出に統一

---

## 回答後の確認

全て回答したら「**回答完了**」または「**done**」とお伝えください。回答に基づいて矛盾を解消する修正を各ドキュメントに反映します。


---

## Q1 回答への補足（AIから工数見積もり）

ユーザー質問「(a)を実現してみたいが、どれくらい工数負荷がかかりそうか」への回答:

### iOS ロック画面ウィジェット工数
- WidgetKit 初期設定: 0.5日
- Widget 本体実装（TimelineProvider + View + ペルソナ顔）: 1〜2日
- ディープリンク設計: 0.5日
- PWA→Capacitor 移行: 1日 →PWAにしないんじゃないの？
- Apple Developer 登録・証明書: 0〜1日（$99/年必須）→不要
- 実機テスト: 1日
- App Store 申請審査: 1〜7日（TestFlight推奨）
- **iOS 小計: 約4〜5日（審査期間除く）**

### Android ロック画面ウィジェット工数
- AppWidgetProvider 実装: 1〜2日
- Android 12+ ロック画面対応: 0.5日
- ディープリンク設計: 0.5日
- Capacitor Android移行: 0.5日
- Play Store 内部テスト配布: 1日
- **Android 小計: 約3〜4日**

### 判断材料
- **書類審査（5/10）対象**: aidlc-docs のみ。ウィジェット実機は不要
- **予選プレゼン**: iOS 一本で TestFlight デモで十分
- **決勝（AWS実デプロイ）**: 数週間期間、Android 追加対応可能
- **チーム3人並行**: iOS担当＋Android担当＋PWA/サーバー担当

### Q1 最終確認

上記を踏まえて、最終方針は以下で進めてよいですか？

A) **書類審査用の設計には (a) を含める**。実装工数を明記し、予選プレゼン時に iOS TestFlight で実機デモ、決勝で Android 追加
B) **書類審査でも (a) を外して PWA 一本に絞る**（安全路線）
C) **MVPはPWA、(a) は決勝で追加（最初から含めない）**
X) Other

[Answer]: A
