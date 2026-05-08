# Components

gutitto のシステムを構成する主要コンポーネントを定義する。

## コンポーネント一覧

```mermaid
flowchart TB
    subgraph Entry["⚡ 入口レイヤー"]
        NativeApp[Native App Shell<br/>Capacitor iOS/Android]
        Widget[Lockscreen/Home Widget<br/>WidgetKit / AppWidgetProvider]
        CallUI[Call UI<br/>Web (HTML/CSS/JS)]
    end
    
    subgraph Core["🫂 会話コアレイヤー"]
        VoiceProxy[Voice Session Manager<br/>音声セッション管理]
        AcceptEngine[Accepting Engine<br/>受容プロンプト構築]
        PersonaManager[Persona Manager<br/>ペルソナ適用]
        AizuchiEngine[Aizuchi Engine<br/>相槌・自発発話制御<br/>クライアント側VAD]
    end
    
    subgraph Knowledge["🧠 YouKnowMe レイヤー"]
        PreviewAgent[Preview Agent<br/>予習エージェント（通話前実行）]
        MemoryStore[Memory Store<br/>階層記憶（S3 + JSON）]
        StructMem[Structured Memory<br/>DynamoDB（簡易）]
        DataConnectors[Data Connectors<br/>Calendar/Slack/Gmail/会話履歴]
    end
    
    subgraph Output["📝 整理・配信レイヤー"]
        AnalysisEngine[Analysis Engine<br/>会話分析・マトリクス分類]
        DeliveryRouter[Delivery Router<br/>マルチチャネル配信]
    end
    
    subgraph External["☁️ 外部サービス"]
        GrokAPI[Grok Voice API<br/>xAI Realtime]
        Bedrock[Amazon Bedrock<br/>要約・分析・抽出]
        GoogleAPI[Google APIs<br/>Calendar/Gmail]
        SlackAPI[Slack API]
    end
    
    NativeApp --> CallUI
    Widget --> NativeApp
    CallUI --> VoiceProxy
    
    VoiceProxy --> GrokAPI
    VoiceProxy --> AcceptEngine
    AcceptEngine --> PersonaManager
    AcceptEngine --> AizuchiEngine
    AcceptEngine --> PreviewAgent
    
    PreviewAgent --> DataConnectors
    PreviewAgent --> Bedrock
    PreviewAgent --> MemoryStore
    PreviewAgent --> StructMem
    DataConnectors --> GoogleAPI
    DataConnectors --> SlackAPI
    
    VoiceProxy -->|通話終了| AnalysisEngine
    AnalysisEngine --> Bedrock
    AnalysisEngine --> DeliveryRouter
    DeliveryRouter --> SlackAPI
```

---

## コンポーネント詳細

### 1. Native App Shell / Call UI / Widget（入口レイヤー）

| 項目 | 内容 |
|------|------|
| **責務** | Capacitor ハイブリッドネイティブアプリとして iOS/Android で動作し、ロック画面/ホーム画面ウィジェットから通話画面への起動を提供 |
| **3本柱** | ⚡ ゼロUX |
| **主要機能** | ロック画面/ホーム画面ウィジェット、1タップ起動、ネイティブディープリンク、WebView による通話UI表示、音声再生（AudioContext）、音声認識（Web Speech API）、クライアント側VAD（相槌タイミング検出）、設定画面（奥側） |
| **依存先** | Voice Session Manager（WebSocket接続） |
| **技術** | **Capacitor**（フロント: HTML/CSS/JS）、iOS WidgetKit（Swift）、Android AppWidgetProvider（Kotlin）、TestFlight / Play Store 配布 |
| **MVP方針** | iOS 版を TestFlight で予選プレゼン向けに配布。Android 版は決勝フェーズ |

### 2. Voice Session Manager（会話コア）

| 項目 | 内容 |
|------|------|
| **責務** | クライアントと音声AIプロバイダ間のリアルタイム音声セッションを管理。プロキシ＋セッション初期化＋ペルソナ設定注入 |
| **3本柱** | 🔄 横断（全柱を接続する中核） |
| **主要機能** | WebSocket確立、Grok Realtime API接続（Bearer認証）、session.update（ペルソナ設定注入）、音声ストリーム双方向転送、通話開始/終了イベント発火、マルチプロバイダ抽象化インターフェース |
| **依存先** | Grok Voice API（外部）、Accepting Engine、Preview Agent（プロンプト注入用） |
| **技術** | AWS Lambda（API Gateway WebSocket）、Node.js + ws |

### 3. Accepting Engine（会話コア）

| 項目 | 内容 |
|------|------|
| **責務** | 受容応答の**事前プロンプト構築**に特化。整理強要ゼロ・無条件受容・寄り添い応答のルールを Grok の instructions に注入する |
| **3本柱** | 🫂 受け止める力 |
| **主要機能** | システムプロンプト構築（受容ルール＋ペルソナ設定＋予習情報を統合）、感情トーン検出（プロンプト調整目的）、ログ記録（KPI改善のための観測目的。事後検証・抑制は行わない） |
| **依存先** | Persona Manager、Preview Agent |
| **技術** | プロンプトエンジニアリング（Grok session.update の instructions フィールド）、Bedrock（補助分析） |
| **設計判断** | 事後検証（validateResponse）は実時間で抑制不可能なため削除。**事前プロンプトのみ**で品質を担保する |

### 4. Persona Manager（会話コア）

| 項目 | 内容 |
|------|------|
| **責務** | ペルソナ定義の管理・適用。JSON形式のペルソナデータを読み込み、会話セッションに反映する |
| **3本柱** | 🔄 横断 |
| **主要機能** | ペルソナ CRUD（DynamoDB）、ペルソナ切替、ペルソナ設定の session.update 変換、声・温度・VAD設定の適用 |
| **依存先** | DynamoDB（ペルソナ格納） |
| **技術** | DynamoDB |

### 5. Aizuchi Engine（会話コア）

| 項目 | 内容 |
|------|------|
| **責務** | 相槌と自発発話のタイミング・内容制御。**クライアント側で沈黙検出**し、相槌だけ/話題提供/静かに待つ を段階的に選択する |
| **3本柱** | 🫂 受け止める力 ＋ 🧠 YouKnowMe |
| **主要機能** | **クライアント側VAD**で沈黙深度検出（数秒→5秒→8秒の段階）、文脈分析による相槌カテゴリ選択（positive/negative/surprise等）、予習ベース話題提供、話題クールダウン管理、相槌連続回避 |
| **依存先** | Preview Agent（話題候補取得）、Structured Memory（継続話題・人物情報） |
| **技術** | クライアント側 JS（Web Audio API による無音検出） + サーバー Lambda（相槌API）+ Bedrock（文脈分析補助） |

### 6. Preview Agent（YouKnowMe）

| 項目 | 内容 |
|------|------|
| **責務** | **通話前トリガー**でユーザー情報を収集・要約する。会話処理から独立した専用エージェント |
| **3本柱** | 🧠 あなたを知っていること |
| **主要機能** | 通話前トリガー実行（定期実行はしない）、Bedrock による要約・エンティティ抽出、短期記憶への結果保存、会話開始時のプロンプト注入（直近＋関連情報を選択） |
| **依存先** | Data Connectors、Memory Store、Structured Memory、Bedrock |
| **技術** | AWS Lambda（通話前トリガー）、Amazon Bedrock |
| **設計判断** | 定期実行は不要（ユーザーが通話しない限り予習結果を使わないため）。NFR-Performance-3（5秒以内）で完了 |

### 7. Memory Store（YouKnowMe）

| 項目 | 内容 |
|------|------|
| **責務** | 階層記憶の永続化と検索。**ペルソナ別に会話履歴を分離**。短期/中期/長期の3層で情報を保持 |
| **3本柱** | 🧠 あなたを知っていること |
| **主要機能** | 短期記憶保存（会話サマリ、90日TTL）、中期記憶（30〜90日のサマリ）、長期記憶（要約＋メタデータのみ恒久保存、生データは保存しない）、**ペルソナ別プレフィックス**（`{userId}/{personaId}/...`）、キーワード検索 |
| **依存先** | S3（暗号化保存） |
| **技術** | S3 + JSON（MVP）。決勝フェーズで Vector DB（Pinecone / pgvector）とRAGベクトル検索を追加 |
| **設計判断** | MVP ではベクトル検索を使わない（コストと複雑度を抑制）。キーワード検索＋Bedrockプロンプト詰めで十分成立する |

### 8. Structured Memory（YouKnowMe）

| 項目 | 内容 |
|------|------|
| **責務** | ユーザーの人物・イベント・タスクなどを構造化して保持する軽量ストア（「Knowledge Graph」の MVP 版） |
| **3本柱** | 🧠 あなたを知っていること |
| **主要機能** | エンティティ管理（人物・イベント・タスク）、話題候補抽出、"私だけが知ってる" マーキング |
| **依存先** | DynamoDB |
| **技術** | DynamoDB（MVP: フラットなエンティティ構造）。決勝で隣接リスト形式 or Amazon Neptune に拡張 |
| **設計判断** | MVP では関係性エッジや指示語解釈の本格実装は省略（Bedrock プロンプトで十分に対応可能）。コンポーネントとしての存在は残すが、実装は最小限 |

### 9. Data Connectors（YouKnowMe）

| 項目 | 内容 |
|------|------|
| **責務** | 外部データソースとの接続・認証・データ取得を抽象化する |
| **3本柱** | 🧠 あなたを知っていること |
| **主要機能** | Google Calendar OAuth + イベント取得、Slack OAuth + メッセージ取得、Gmail readonly + メール取得、過去会話履歴読み込み、OAuth トークンリフレッシュ管理 |
| **依存先** | Google APIs、Slack API、Secrets Manager（OAuth Secrets） |
| **技術** | AWS Lambda、Secrets Manager |
| **備考** | Google OAuth（Calendar / Gmail readonly）の同意画面は**初回セットアップ時のみ**の摩擦として許容する（ゼロUXは「毎回の利用」に対する原則） |

### 10. Analysis Engine（整理・配信）

| 項目 | 内容 |
|------|------|
| **責務** | 通話終了後に会話全文を分析し、仕事/プライベート × 報連相/タスク/願望のマトリクスに分類する |
| **3本柱** | 🫂 受け止める力（R-Accept-6: 整理は裏で） |
| **主要機能** | 会話全文の Bedrock 分析、マトリクス分類、重要度・緊急度判定、アクションプラン生成、翌日リマインダー生成 |
| **依存先** | Bedrock、Memory Store（会話ログ取得）、Delivery Router |
| **技術** | AWS Lambda（通話終了イベントトリガー）、Amazon Bedrock |

### 11. Delivery Router（整理・配信）

| 項目 | 内容 |
|------|------|
| **責務** | 分析結果をユーザーが指定した配信先に自動投稿する。マルチチャネル対応 |
| **3本柱** | 🫂 受け止める力（ゼロUX: 確認を求めない） |
| **主要機能** | 配信先設定管理（仕事用/プライベート用を別々に）、Slack Webhook投稿（**個人用workspace前提**）、LINE Notify投稿、メール送信、カレンダーToDo追加、翌朝リマインダースケジュール、フォールバック（アプリ内ダッシュボード蓄積） |
| **依存先** | Slack API、LINE API、Gmail API、Google Calendar API |
| **技術** | AWS Lambda + EventBridge（スケジュール配信） |

---

## コンポーネントとストーリーの対応

| コンポーネント | 対応 Must ストーリー | 対応 Should/Could ストーリー |
|-------------|-------------------|--------------------------|
| Native App Shell / Call UI / Widget | 1.1 | 1.2, 1.3, 2.6, 2.7 |
| Voice Session Manager | 2.1, 2.2, 2.3 | 2.7 |
| Accepting Engine | 2.2 | — |
| Persona Manager | — | 2.6 |
| Aizuchi Engine | 2.3 | — |
| Preview Agent | 2.1, 4.1 | 2.4, 2.5, 4.2, 4.3 |
| Memory Store | 4.1 | 2.5, 4.2 |
| Structured Memory | 4.1 | 2.4, 2.5 |
| Data Connectors | 4.1 | — |
| Analysis Engine | — | 3.1 |
| Delivery Router | — | 3.2 |

---

## MVP スコープの簡略化ポイント（予選向け）

| 元の設計 | MVP簡略化 | 理由 |
|---------|---------|-----|
| Knowledge Graph（DynamoDB隣接リスト + エッジ管理） | **Structured Memory**（フラットなエンティティ格納のみ） | 予選ではBedrockプロンプトで十分。グラフ構造は決勝追加 |
| Vector DB（OpenSearch Serverless） | **S3 + JSON + キーワード検索** | コスト ($350/月) 削減、シンプル化 |
| Preview Agent 定期実行（1時間ごと） | **通話前トリガーのみ** | コスト・複雑度削減 |
| Accepting Engine validateResponse | **事前プロンプトのみ** | 実時間抑制は技術的に不可能 |
| 長期記憶全文恒久保存 | **要約＋メタデータのみ恒久保存** | 長期コスト抑制、プライバシー配慮 |
| Siri/Google Assistant カスタムショートカット | **決勝フェーズで追加** | App Intents ネイティブ実装が必要 |
