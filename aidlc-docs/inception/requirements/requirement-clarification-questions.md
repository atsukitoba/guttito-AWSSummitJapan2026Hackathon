# Requirements Clarification Questions

前回の質問への回答を確認したところ、以下4点に曖昧さ・未回答がありました。
次のステージ（要件ドキュメント生成）へ進む前に確定させたいので、回答をお願いします。

---

## Clarification 1: Q4「通話インターフェース」が "AまたはD"

**元の回答**: 「AまたはD  ゼロUXを徹底。特定のアプリを開いてログインではなく、誰かに連絡をとる感覚で」

AとDはまったく違う実装パスになります：
- A: LINE通話連携（既存アプリ「LINE」の中から電話）
- D: iOS/Androidネイティブ（独自アプリを開く、ただしPWA含む）

"ゼロUX（誰かに連絡する感覚）" を最重視すると A が最も近いですが、決勝時のAWSデプロイ（Q10でフルサーバーレス）との相性や技術難易度も関係します。

### 1-1: MVP（予選デモ）段階での通話インターフェース

A) **LINE通話連携一本** — LINE Messaging API + VoIP。ゼロUX徹底だが実装は高難度（審査・費用承認も必要）
B) **PWA（Progressive Web App）** — スマホのホーム画面から1タップ起動。ブラウザベースだが"アプリっぽい"体験
C) **Web App 一本** — ブラウザでボタンタップ → WebSocket 通話で最短で動くものを用意
D) **MVP時はWeb App/PWA、決勝時にLINE連携を追加** — 段階的アプローチ
X) Other (please describe after [Answer]: tag below)

[Answer]: 
ゼロUXを実現するために他に何か手段がありますか？think hard
手段よりも頭のリソースを使わない、待たせない、そばにいる人のように電話や連絡をする感覚が満たせていればいいです
---

## Clarification 2: Q9「プライバシー・データ保護」が未回答

本音の一次情報（愚痴・悩み）を扱うため、データ保護は審査員も気にするポイントです。

A) 最小限 — 会話ログはS3にユーザーID付きで保存、暗号化なし
B) 標準 — S3暗号化 + 個人データは個別バケット + TTL設定
C) 堅牢 — S3暗号化 + KMS + ユーザー削除UI + データエクスポート権
D) 最堅牢 — C + ローカル保存オプション + 完全ユーザーコントロール
E) **MVPはB、将来Cへ段階移行** — 予選段階は標準、決勝・本番はより堅牢
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## Clarification 3: Q13「Grok API依存」が C選択＋"grok" と矛盾

**元の回答**: 「C  grok」

- C = マルチプロバイダ（Grok / OpenAI / Gemini 切替可能）
- "grok" = Grokのみ使う？

真意はどちらですか？

A) **Grok一本で勝負** — Grokの独自性（2025/12リリースの新鋭API）を差別化として押し出す。フォールバックなし
B) **Grokメイン＋最小フォールバック** — 通常はGrok、API障害時のみPolly/Transcribeに切替
C) **マルチプロバイダ切替** — ペルソナや気分で Grok / OpenAI Realtime を選べるようにする
D) **MVPはGrok一本、設計上は差替え可能に** — 実装はGrokのみだが、内部インターフェースは抽象化して将来差替え可能
X) Other (please describe after [Answer]: tag below)

[Answer]: C

---

## Clarification 4: Q15「Unit分解の観点」が未回答 ★最重要

審査員は "なぜこう分けたか" を重視するため、Unit分解の軸は**書類審査の生命線**です。

他の回答（Q3:予習付き、Q10:フルサーバーレス、Q11:継続率KPI、Q12:終了後フィードバック重視）を踏まえると、**ユーザージャーニー軸＋ビジネス価値軸**が説明性高いと推薦します。

A) **ユーザージャーニー軸** — ①予習 ②会話体験 ③分析・フィードバック の3Unit
B) **責務軸（データフロー）** — 入力層 / コア処理 / 出力層
C) **並行開発しやすさ軸** — チーム3人（鳥羽/平湯/松井）が独立作業できる粒度
D) **ビジネス価値軸** — MVPコア / 予習機能 / 会話後フィードバック など価値別
E) **AWSサービス境界軸** — Connect、Lambda群、Bedrock、S3 など
F) **A + D のハイブリッド** — ユーザージャーニーを軸にしつつ、各UnitがMVP/Nice-to-Have価値を含む構成
X) Other (please describe after [Answer]: tag below)

[Answer]: 
F
---

## 追加の確認: ビジネス構想（プレゼンスライド8準拠）

PRESENTATIONのアーキテクチャ図では以下4層が描かれています:
- 📱 Input Layer（iOS/Android App）
- 🎙️ Voice Agent（Grok Voice Agent API）
- 📋 Context Preparation（Lambda + Google/Slack API + Bedrock要約 + S3）
- 📝 Text Generation（Lambda + Bedrock分析 + S3）
- 📤 Output Layer（Lambda + Slack Webhook）

### Add-1: このアーキテクチャ4層構造を Inception / Construction の Unit 構成にそのまま採用しますか？

A) そのまま採用 — 4層をそのまま4Unit（または3Unit+output統合）として分割
B) 採用するが名称リネーム — 審査員に伝わりやすい名称に変更（例: 予習Unit / 会話Unit / 整理Unit / 配信Unit）
C) 独自軸で再構成 — スライド図はプレゼン用、Unit分割は別軸で設計
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## 回答後の確認

全て回答したら「**回答完了**」または「**done**」とお伝えください。回答後、requirements.md を生成します。


---

## Clarification 1 (Re-confirm): ゼロUX実現の具体手段

"think hard" で手段を網羅的に検討しました。ゼロUX（頭リソース使わない / 待たせない / そばにいる人のように電話する感覚）を満たす選択肢:

**電話系（真のゼロUX）**:
- P1: Amazon Connect — スマホ連絡先に"まなみ"登録→標準電話アプリから発信
- P2: LINE通話連携

**音声アシスタント系**:
- P4: Siri/Google Assistantショートカット — 「Hey Siri、まなみに電話」

**アプリ系**:
- P7: PWA — ホーム画面アイコンから1タップ
- P8: ウィジェット — ホーム画面常駐の大ボタン

**ハイブリッド**:
- P11: **Amazon Connect ＋ PWA** ★推薦
  - 予選デモ: PWA で動作（現行Web路線を拡張、最短で動く）
  - 決勝: Amazon Connect で電話番号発行 → 連絡先登録 → 標準電話アプリから発信（真のゼロUX）
  - ハッカソンテーマ「人をだめにする」に最強に刺さる
  - AWS Summit と親和性◎
- P12: Siriショートカット ＋ PWA（iPhone前提で最小コスト）

### どれで進めますか？

A) P11（Amazon Connect ＋ PWA のハイブリッド）★AI推薦
B) P1単独（Amazon Connect のみ、最初から電話番号）
C) P7単独（PWAのみ、決勝も同じ）
D) P12（Siriショートカット ＋ PWA）
E) その他・複数組合せ
X) Other (please describe after [Answer]: tag below)

[Answer]: 


---

## Clarification 1 (Re-confirm v2): "そばにいる・理解している" UI

「電話帳登録は古い」というフィードバックを受けて再設計。本質は **"AI側から話しかけてくる" × "常に視界の隅にいる"** と捉え直しました。

### 選択肢

**A) P-NEW（推薦）** — "AI主導 ＋ アンビエント常駐" モデル（3要素の多層体験）
  - ①PWA常駐（ホーム画面 ＋ ロック画面ウィジェット）
  - ②AI能動プッシュ（カレンダー/時間/位置をトリガーにキャラから「話す？」）
  - ③通知タップで即会話（ロック解除不要、1タップで音声起動）
  - 予選: PWA + Web Push + Notification Actions
  - 決勝: + AWS SNS/IoT + iOS Live Activity + Apple Watch対応

**B) AI主導プッシュのみに絞る** — AIから能動的に通知 → 通知タップで即会話（ウィジェットは後回し）

**C) アンビエント常駐のみに絞る** — ホーム画面ウィジェット＋ロック画面が主、通知起動はサブ

**D) ウェアラブル/AirPods特化** — Apple Watch・AirPods長押しで即起動、耳元体験を最重視

**E) Siri/音声アシスタント経由** — 「Hey まなみ」で呼び出し。ハンズフリー性最重視

**F) 複数組合せ（具体的に指定）**

X) Other (please describe after [Answer]: tag below)

[Answer]: 
C

---

## Clarification 1 (Final Understanding): 本質は"受け止める力"

### 本質の再定義（ユーザー指摘を反映）

gutittoの真の価値は **入口UI ではなく、受け止める側（AI振る舞い・ペルソナ）** にある。

> 「大人として感情の整理や思考の整理をせずとも、気持ちや思いをぶつけても全て受け止めてくれる存在がいること」

これを軸に要件を組み直す。

### 受け止める力を構成する要件（要件ドキュメントに反映）

- **R-Accept-1**: 無条件受容 — 整理されていない愚痴・矛盾する発言・暗い感情を否定せず受け止める
- **R-Accept-2**: 整理強要ゼロ — 「それって何の話？」「もっと具体的に」など構造化要求を絶対にしない
- **R-Accept-3**: 大人疲労の理解 — ユーザーが日中張り詰めた緊張をほどく方向に作用する
- **R-Accept-4**: あなたを知っている感 — 自己紹介不要、過去を覚えている、予習済み
- **R-Accept-5**: 寄り添う応答 — 単なる相槌ではなく気持ちに寄り添う応答（「うん、わかる」「それは疲れるよね」）
- **R-Accept-6**: 整理は会話後に裏で — ユーザーの前では整理させず、会話後にBedrockがサマリ化

### 入口UIについての最終方針

入口は **最小摩擦** で十分（引き算設計）。本質的な"受容"に影響しない範囲で軽くする。

### 確認

以下で進めてよいですか？

A) **PWA + 1タップ大ボタン起動**。AI能動プッシュ／ウィジェット等は決勝での上乗せ要素として計画。予選デモは最小摩擦入口で "受け止める力" をプロンプト・ペルソナ・相槌で主張する ★AI推薦
B) 上記＋予選時点でAI能動プッシュも必須要件に含める
C) 上記＋予選時点でウィジェット常駐も必須要件に含める
D) 他の方向性
X) Other (please describe after [Answer]: tag below)

[Answer]: A
