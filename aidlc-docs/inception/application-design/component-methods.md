# Component Methods

各コンポーネントの主要メソッド（インターフェース）を定義する。
詳細なビジネスロジックは Construction フェーズの Functional Design で設計する。

---

## Voice Session Manager

| メソッド | 入力 | 出力 | 責務 |
|---------|------|------|------|
| `startSession(userId, personaId)` | ユーザーID, ペルソナID | SessionHandle | Grok API接続確立、ペルソナ設定注入、予習コンテキスト注入 |
| `endSession(sessionId)` | セッションID | ConversationLog | WebSocket切断、会話ログ確定、後処理パイプライン起動 |
| `forwardAudio(sessionId, audioChunk)` | セッションID, PCM16データ | void | クライアント→Grok 音声転送 |
| `forwardResponse(sessionId, grokMessage)` | セッションID, Grokレスポンス | void | Grok→クライアント 応答転送 |
| `injectContext(sessionId, context)` | セッションID, 予習コンテキスト | void | session.update で instructions を更新 |

---

## Accepting Engine

| メソッド | 入力 | 出力 | 責務 |
|---------|------|------|------|
| `buildSystemPrompt(persona, previewContext, acceptRules)` | ペルソナ設定, 予習情報, 受容ルール | string (prompt) | 統合システムプロンプト構築 |
| `detectEmotionTone(conversationContext)` | 会話コンテキスト | ToneSignal | 感情トーン検出（プロンプト調整目的） |
| `logResponseQuality(aiResponse, context)` | AI応答テキスト, 会話コンテキスト | void | 応答品質のログ記録（KPI改善のための観測目的のみ。実時間抑制はしない） |

**設計判断**: 事後の `validateResponse` による抑制は実時間では不可能なため削除。**事前プロンプトのみ**で品質を担保する。

---

## Persona Manager

| メソッド | 入力 | 出力 | 責務 |
|---------|------|------|------|
| `getPersona(personaId)` | ペルソナID | PersonaConfig | ペルソナ設定取得 |
| `listPersonas(userId)` | ユーザーID | PersonaConfig[] | ペルソナ一覧取得 |
| `switchPersona(userId, personaId)` | ユーザーID, ペルソナID | void | アクティブペルソナ切替 |
| `toSessionConfig(persona)` | PersonaConfig | GrokSessionUpdate | ペルソナ→Grok session.update 変換 |

---

## Aizuchi Engine

| メソッド | 入力 | 出力 | 責務 |
|---------|------|------|------|
| `onSilenceDetected(silenceDuration, context)` | 沈黙時間(ms), 会話コンテキスト | AizuchiAction | 段階的応答選択（待つ/相槌/話題提供） |
| `selectAizuchi(emotionCategory, history)` | 感情カテゴリ, 直近相槌履歴 | string (phrase) | 相槌フレーズ選択（連続回避） |
| `selectTopic(knowledgeGraph, recentTopics)` | 知識グラフ, 直近話題 | string (topic) | 自発発話話題選択（クールダウン考慮） |

---

## Preview Agent

| メソッド | 入力 | 出力 | 責務 |
|---------|------|------|------|
| `runPreview(userId)` | ユーザーID | PreviewResult | **通話前トリガーで実行**。全データソースから情報収集→Bedrock要約→Structured Memory 更新 |
| `getContextForSession(userId, personaId)` | ユーザーID, ペルソナID | SessionContext | 通話開始時のプロンプト注入用コンテキスト生成（ペルソナ別会話履歴＋共通予習情報） |
| `resolveReference(phrase, userId)` | 指示語（「あの人」等）, ユーザーID | ResolvedEntity | 文脈ショートカット解釈（MVPは簡易実装） |
| `markLongTermMemory(utterance, category)` | 発話テキスト, カテゴリ(秘密/願望/夢) | void | 長期記憶マーキング（要約のみ保存） |

---

## Memory Store

| メソッド | 入力 | 出力 | 責務 |
|---------|------|------|------|
| `saveShortTerm(userId, personaId, data)` | ユーザーID, ペルソナID, 会話サマリ | void | 短期記憶保存（ペルソナ別プレフィックス） |
| `saveMidTerm(userId, personaId, summary)` | ユーザーID, ペルソナID, サマリ | void | 中期記憶保存 |
| `saveLongTerm(userId, personaId, episode)` | ユーザーID, ペルソナID, エピソード要約 | void | 長期記憶保存（要約＋メタデータのみ、生データ保存せず） |
| `search(userId, personaId, query)` | ユーザーID, ペルソナID, 検索クエリ | MemoryResult[] | キーワード検索（MVP）。決勝でベクトル検索拡張 |
| `getRecent(userId, personaId, n)` | ユーザーID, ペルソナID, 件数 | MemoryResult[] | 直近N件取得 |

**設計判断**: ペルソナ別に会話履歴を分離（共通データ＝予習情報はペルソナ間で共有）。MVPはキーワード検索、決勝でベクトル検索を追加。

---

## Structured Memory

| メソッド | 入力 | 出力 | 責務 |
|---------|------|------|------|
| `addEntity(userId, entity)` | ユーザーID, エンティティ(人物/イベント/タスク) | EntityId | エンティティ追加 |
| `getEntities(userId, type)` | ユーザーID, エンティティ種別 | Entity[] | 種別別にエンティティ取得 |
| `resolveAmbiguity(userId, phrase, candidates)` | ユーザーID, 指示語, 候補リスト | Entity | 最確度エンティティ推定（MVPは簡易版、決勝で精度向上） |
| `getTopicCandidates(userId, limit)` | ユーザーID, 件数上限 | Topic[] | 自発発話用話題候補取得 |

**設計判断**: MVP では関係性エッジを省略し、エンティティ管理のみに絞る（決勝で Knowledge Graph 本格化: 隣接リスト or Neptune）。

---

## Data Connectors

| メソッド | 入力 | 出力 | 責務 |
|---------|------|------|------|
| `fetchCalendarEvents(userId, range)` | ユーザーID, 日付範囲 | CalendarEvent[] | Google Calendar イベント取得 |
| `fetchSlackMessages(userId, channels, limit)` | ユーザーID, チャンネル, 件数 | SlackMessage[] | Slack メッセージ取得 |
| `fetchEmails(userId, filter)` | ユーザーID, フィルタ条件 | Email[] | Gmail 未読/重要メール取得 |
| `refreshToken(userId, provider)` | ユーザーID, プロバイダ名 | void | OAuth トークンリフレッシュ |

---

## Analysis Engine

| メソッド | 入力 | 出力 | 責務 |
|---------|------|------|------|
| `analyzeConversation(conversationLog)` | 会話ログ全文 | AnalysisResult | Bedrock でマトリクス分類 |
| `generateActionPlan(analysisResult)` | 分析結果 | ActionPlan | アクションプラン生成 |
| `generateDailyReport(analysisResult)` | 分析結果 | DailyReport | 日報サマリ生成 |

---

## Delivery Router

| メソッド | 入力 | 出力 | 責務 |
|---------|------|------|------|
| `deliver(userId, content, channel)` | ユーザーID, 配信内容, チャネル設定 | DeliveryResult | 指定チャネルへ配信 |
| `scheduleReminder(userId, content, time)` | ユーザーID, 内容, 配信時刻 | void | 翌朝リマインダー設定 |
| `getDeliveryConfig(userId)` | ユーザーID | DeliveryConfig | 配信先設定取得（仕事用/プライベート用） |
