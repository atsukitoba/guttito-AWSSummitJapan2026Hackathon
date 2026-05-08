# Unit of Work

gutitto の開発作業単位（Unit of Work）を定義する。

---

## なぜこう分けたか（Unit 分解の基本思想）

### 分解軸

**ユーザージャーニー軸 × 3本柱軸 のハイブリッド**（Requirements Q15: F の回答に基づく）

```
          ユーザージャーニー軸
     ┌────────────┬────────────┬────────────┐
     │  Pre-call  │  In-call   │ Post-call  │
     │  (予習)    │  (受容)    │  (整理)    │
     └────────────┴────────────┴────────────┘
              ×
          3本柱（ビジネス価値軸）
     ┌────────────────────────────────────┐
     │ 柱1: 受け止める力（R-Accept）      │
     │ 柱2: ゼロUX（R-Entry）             │
     │ 柱3: あなたを知っていること         │
     │      （R-YouKnowMe）               │
     └────────────────────────────────────┘
```

### 分解判断の5つの原則

1. **各Unitが単一のジャーニー局面と、単一の柱を主に担う** — 説明性が高い
2. **チーム3名（鳥羽・平湯・松井）で並行開発可能な粒度** — ハッカソン期間内の並列作業前提
3. **MVP Must 5本を最小コンポーネントで動かせる** — 実現可能性重視
4. **審査員への "なぜこう分けたか" が自明** — 書類審査の生命線
5. **各Unit間のインターフェースが明確** — 結合度低く保つ

### 代替案を選ばなかった理由

| 代替案 | 不採用の理由 |
|-------|------------|
| 責務軸（Input/Core/Output の3層） | ユーザー価値が見えない。「どう作るか」の話で「なぜ作るか」を語れない |
| AWSサービス境界（Lambda/DynamoDB/S3...） | 技術視点のみ。プロダクト価値と切り離れる |
| 機能フラット列挙（1機能1Unit） | 粒度が細かすぎて、チーム3人での並行作業の単位になりにくい |
| モノリシック単一Unit | 並行開発不可、役割分担不明確 |

---

## Unit 一覧

gutitto は **4つの Unit** で構成する。

| Unit ID | Unit 名 | ジャーニー | 主軸の柱 | MVP構成要素 | 担当候補 |
|---------|--------|----------|---------|----------|--------|
| U1 | **入口 Unit**（Entry Unit） | Pre-call | ⚡ ゼロUX | Native App Shell, Widget, Call UI | フロント担当 |
| U2 | **会話 Unit**（Conversation Unit） | In-call | 🫂 受け止める力 | Voice Session Manager, Accepting Engine, Persona Manager, Aizuchi Engine | バックエンド担当 |
| U3 | **予習 Unit**（Preview Unit） | Pre-call + In-call | 🧠 あなたを知っていること | Preview Agent, Data Connectors, Memory Store, Structured Memory | AI統合担当 |
| U4 | **整理・配信 Unit**（Output Unit） | Post-call | 🫂 受け止める力（裏側） | Analysis Engine, Delivery Router | 決勝時に全員で |

---

## Unit 詳細

### U1: 入口 Unit — 「頭も心も身体も使わせない」の実装

**責務**:
- Capacitor ハイブリッドネイティブアプリとしての iOS/Android ビルド
- ロック画面/ホーム画面ウィジェット（iOS WidgetKit / Android AppWidgetProvider）
- ウィジェットからの1タップ起動（ロック解除不要）
- 通話画面UI（巨大な通話ボタン1つ、ペルソナの顔、引き算UI）
- 音声入出力（AudioContext、Web Speech API）
- クライアント側VAD（相槌タイミング検出）
- ログイン永続化（リフレッシュトークン管理）

**含むコンポーネント**:
- Native App Shell（Capacitor）
- Widget Extension（iOS WidgetKit / Android AppWidgetProvider）
- Call UI（Web HTML/CSS/JS）

**MVP 成功基準**:
- [ ] iOS 版が TestFlight で配布可能（予選プレゼン向け）
- [ ] ロック画面ウィジェットから 1タップで通話画面が起動する
- [ ] 通話画面に通話ボタン以外の操作要素がない
- [ ] 再訪時にログイン画面が出ない
- [ ] NFR-Performance-2（3秒以内起動）達成

**主なストーリー**: 1.1, 1.2, 1.3, 2.7

**主な柱への貢献**: ⚡ ゼロUX（プロダクトの差別化ポイント#2）

**決勝フェーズ追加**: Android 版、ロック画面クイックアクション、Siri/Google Assistant ショートカット、Live Activity、Apple Watch コンパニオン

---

### U2: 会話 Unit — 「整理しなくても受け止めてくれる」の実装

**責務**:
- WebSocket プロキシによる Grok Voice API 接続管理
- ペルソナ設定の適用（session.update）
- 受容応答プロンプト構築（Accepting Engine）
- 相槌・自発発話のタイミング・内容制御（Aizuchi Engine）
- ペルソナ CRUD・切替
- マルチプロバイダ抽象化インターフェース

**含むコンポーネント**:
- Voice Session Manager
- Accepting Engine
- Persona Manager
- Aizuchi Engine

**MVP 成功基準**:
- [ ] ユーザー発話 → AI応答の往復が成立
- [ ] ペルソナ（仮名: まなみ／げんぞう／＋1、確定時に変更）で切替可能、各キャラクターで応答が変わる
- [ ] 整理されていない愚痴を否定せず受容する応答が返る
- [ ] 沈黙時に相槌 or 話題提供が選択される
- [ ] NFR-Performance-1（MVP 1.5秒）を目標に計測

**主なストーリー**: 2.1, 2.2, 2.3, 2.6

**主な柱への貢献**: 🫂 受け止める力（プロダクトの中核）

**決勝フェーズ追加**: マルチプロバイダ切替実装（OpenAI Realtime / Gemini Live）、感情分析による応答トーン動的調整

---

### U3: 予習 Unit — 「あなたを知っている」の実装

**責務**:
- 通話前トリガーでの予習エージェント実行
- 外部データソース連携（Google Calendar / Slack / Gmail / 過去会話）
- OAuth 認証管理とトークンリフレッシュ
- Bedrock による要約・エンティティ抽出
- 階層記憶管理（S3 + JSON、ペルソナ別分離）
- 構造化メモリ（DynamoDB、人物・イベント・タスク）
- 会話セッション起動時のプロンプト注入

**含むコンポーネント**:
- Preview Agent
- Data Connectors
- Memory Store（S3+JSON、ペルソナ別プレフィックス）
- Structured Memory（DynamoDB）

**MVP 成功基準**:
- [ ] Googleカレンダー OAuth 連携が動作し、今日/明日の予定が取得できる
- [ ] Slack OAuth 連携が動作し、直近メッセージが取得できる
- [ ] 通話前トリガーで予習エージェントが発火し、NFR-Performance-3（5秒以内）で完了
- [ ] 予習結果が Grok Voice API のプロンプトに正しく注入される
- [ ] ペルソナ別に会話履歴が分離保存される
- [ ] 会話冒頭でAIが「今日の会議どうだった？」のように予習情報を反映した挨拶をする

**主なストーリー**: 2.1（予習反映）, 2.3（自発発話）, 2.4, 2.5, 4.1, 4.2, 4.3

**主な柱への貢献**: 🧠 あなたを知っていること（プロダクトの差別化ポイント#3）

**決勝フェーズ追加**: Knowledge Graph本格化（隣接リスト / Neptune）、Vector DB 導入（Pinecone/pgvector）、RAGベクトル検索、ウェアラブル・位置情報・天気連携、定期実行バッチ

---

### U4: 整理・配信 Unit — 「話して寝るだけで仕事もプライベートも整理されている」の実装

**責務**:
- 通話終了後の会話分析（Bedrock マトリクス分類）
- 仕事/プライベート × 報連相/タスク/願望の6象限分類
- アクションプラン・日報サマリ生成
- マルチチャネル配信（Slack/LINE/Email/Calendar/Notion/アプリ内）
- 翌朝リマインダースケジュール
- 配信先設定管理（仕事用/プライベート用別々）

**含むコンポーネント**:
- Analysis Engine
- Delivery Router

**MVP 成功基準**:
- [ ] 予選デモでは **モック演出** で十分（実装は決勝フェーズ）
- [ ] 書類審査では設計と Unit 構造が明確であれば OK

**主なストーリー**: 3.1, 3.2

**主な柱への貢献**: 🫂 受け止める力（裏側。R-Accept-6 "整理は会話後に裏で" の実装）

**MVP 扱い**: 予選は **モック** で審査クリア、決勝フェーズでフル実装

**決勝フェーズ追加**: 全実装、個人ダッシュボード、KPI計測、週次/月次レポート自動生成

---

## 横断要件

以下は特定の Unit に閉じない横断要件で、**全Unitで対応が必要**:

| 横断要件 | 扱い |
|---------|------|
| NFR-Zero-UX-1〜4 | 各Unitの実装時に考慮。設計レビューで逸脱チェック |
| R-Security-1〜4 | 各Unitでデータ保存時に SSE-S3、Secrets Manager 利用 |
| 認証・セッション（R-Entry-3） | U1 が主責務、他 Unit は API Gateway のオーソライザ経由 |
| マルチプロバイダ抽象化（NFR-Maintainability-1） | U2 が主責務 |
| Session Orchestrator（services.md） | U2 が担うが、U1/U3/U4 とインターフェース調整 |

---

## Code 組織化戦略（Greenfield前提の推奨）

gutitto は **モノレポ構成** で、Unit 別にディレクトリを分割する:

```
gutitto/
├── packages/
│   ├── native-app/              # U1: 入口 Unit（Capacitor + Widget Extension）
│   │   ├── ios/                 # iOS ネイティブ（WidgetKit + Capacitor）
│   │   ├── android/             # Android ネイティブ（AppWidgetProvider + Capacitor）
│   │   └── web/                 # Web 部分（HTML/CSS/JS）
│   ├── conversation-service/    # U2: 会話 Unit（Lambda群）
│   │   ├── voice-session/
│   │   ├── accepting-engine/
│   │   ├── persona-manager/
│   │   └── aizuchi-engine/
│   ├── preview-service/         # U3: 予習 Unit（Lambda群）
│   │   ├── preview-agent/
│   │   ├── data-connectors/
│   │   ├── memory-store/
│   │   └── structured-memory/
│   └── output-service/          # U4: 整理・配信 Unit（Lambda群、決勝）
│       ├── analysis-engine/
│       └── delivery-router/
├── shared/                      # 共通コード（型定義、ユーティリティ）
│   ├── types/
│   └── utils/
├── infrastructure/              # IaC（AWS CDK / SAM）
└── aidlc-docs/                  # 本 Inception 成果物
```

**理由**:
- Unit ごとに独立したビルド・デプロイが可能
- チーム3人が各 Unit を主担当として並行作業できる
- 共通型定義（Persona スキーマ、API 契約）は `shared/types/` に集約

---

## Unit サマリ

| 観点 | 結果 |
|------|------|
| Unit 数 | **4** |
| MVP 対象 Unit | U1, U2, U3（U4 はモック） |
| 決勝フェーズで本格化 | U4 |
| Must ストーリーカバー | U1(1.1) + U2(2.1, 2.2, 2.3) + U3(4.1) = 5/5 = **100%** |
| チーム並行開発 | 3名で U1/U2/U3 を並行、U4 は決勝時に統合 |
| ジャーニー完全性 | Pre-call / In-call / Post-call 全カバー |
| 3本柱完全性 | ⚡ U1, 🫂 U2+U4, 🧠 U3 = 全柱カバー |
