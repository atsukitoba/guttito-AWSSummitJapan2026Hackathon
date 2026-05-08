# gutitto（ぐちっと）

<p align="center">
  <img src="./assets/hero.png" alt="gutitto - グッとくる、あなたがダメになれる誰かがいる" width="100%"/>
</p>

<p align="center">
  <b>グッとくる、あなたがダメになれる誰かがいる。</b>
</p>

<p align="center">
  <i>「大人として感情の整理も思考の整理も必要ない。気持ちや思いをぶつけても、全て受け止めてくれる存在がそばにいる。」</i>
</p>

---

**プロダクト名 / チーム名の関係**: 本プロダクト「gutitto」は「ぐっとくる」「気持ちがこもる」「吐き出す（ぐち）」を掛けた造語で、疲れた夜に "ぐちる" ように安心して何でも話せる体験を象徴する。

---

## 何をするプロダクトか

**整理疲れ・感情のコントロールに苦しむ社会人に、「ダメになれる存在」を提供する。**

疲れた夜にスマホを触る気力すらないユーザーに対して、gutitto は **ロック画面のウィジェットからワンタップ** で、あなたを予習済みの AI ペルソナに人に言いづらい感情、思い、考えを吐き出す場を提供する。ユーザーは整理も気合も要らず、ぽつぽつ話して寝るだけ。会話後は AI が裏で内容を分析し、仕事の報連相・タスク・やりたいことを自動整理して Slack 等に投稿する。

これは **人を能動的にダメになれる存在を提供すること** であり、「**ダメになれる場所** がそばにある安心感」で感情と行動の両方を off-load する、受容 × ゼロ UX × 予習知の三層プロダクトである。

---

## 体験の 3 段階

### 1. 起動前（Pre-call）— "そばにいる" の常駐

ロック画面のウィジェットに AI ペルソナの顔が常駐している。押し付けがましい通知は出さず、**静かな存在感** だけを放つ。ユーザーは無意識にその存在を視界の隅で感じ、いつでも "呼べる" 安心感の中で過ごす。

- ロック画面 / ホーム画面ウィジェット（iOS WidgetKit / Android AppWidgetProvider）
- ペルソナの顔（ピクセルアート）＋ 短いステータス（例:「今日はお疲れさま」）
- 通話経路は **1 タップ**。ロック解除も、アプリ起動画面もなし

### 2. 通話中（In-call）— "整理しなくていい" の受容

ウィジェットをタップした瞬間、ペルソナから先に話しかけてくる。**ユーザーは最初の一言を考える必要すらない**。

- 予習済み AI がコンテキストを持って挨拶:「今日お疲れさま。四半期レビュー長かったね」
- 整理されていない愚痴・矛盾・沈黙も、**否定せず受容する**
- 沈黙は催促せず、必要に応じて相槌だけで寄り添うか、予習ベースで話題を振る
- 通話終了は通話ボタン 1 つ。引き留めなし

### 3. 通話後（Post-call）— "整理は裏で" のアウトプット

通話が終わったら、ユーザーは寝るだけ。整理は AI が裏側で行う。

- 会話全文を Amazon Bedrock で分析
- 「仕事 × 報連相 / タスク / 願望」「プライベート × 報連相 / タスク / 願望」の 6 象限に分類
- 指定された配信先（Slack / LINE / Email / Calendar / Notion 等）にアクションプランを自動投稿
- 翌朝のリマインダーが届いた状態で目覚める

---

## 3 本柱（プロダクトの差別化）

gutitto は **3 本柱** を積層的に備えることで成立する。どれか 1 つが欠けても「ダメになれる」体験は成立しない。

| # | 柱 | 内容 |
|---|---|------|
| 1 | 🫂 **受け止める力（Accepting Power）** | 整理強要ゼロの無条件受容。AI が裏で整理する |
| 2 | ⚡ **ゼロ UX（Zero-UX）** | 頭も心も身体も使わせない起動体験。ロック画面から 1 タップ |
| 3 | 🧠 **あなたを知っていること（You-Know-Me）** | Google カレンダー・Slack・Gmail・過去会話を予習エージェントが集約 |

```
  3 本柱の積層関係
  ┌──────────────────────────────────────────────┐
  │ 3. あなたを知っていること（You-Know-Me）      │
  │    ↓ 「あなたを知っている」から安心できる     │
  │ 2. ゼロ UX（Zero-UX）                         │
  │    ↓ だから気合なしで起動できる               │
  │ 1. 受け止める力（Accepting Power）            │
  │    ↓ だからダメになっても受け止めてもらえる   │
  │                                              │
  │ = "ダメになれる存在" の完成                   │
  └──────────────────────────────────────────────┘
```

---

## アーキテクチャ概要

| レイヤ | 採用技術 |
|--------|--------|
| フロント | Capacitor ハイブリッドネイティブ（iOS / Android）＋ iOS WidgetKit / Android AppWidgetProvider |
| 通話入口 | ロック画面ウィジェット / ホーム画面ウィジェット / ホーム画面アイコン（将来: Siri 等） |
| API | API Gateway（REST + WebSocket） |
| AI 中核 | **xAI Grok Voice API（Realtime）** ＋ Amazon Bedrock（要約・分析・エンティティ抽出） |
| 受容応答 | Accepting Engine（プロンプトエンジニアリングで整理強要ゼロを担保） |
| 予習 | Preview Agent（通話前トリガーで Google Calendar / Slack / Gmail / 過去会話を集約） |
| 記憶 | 階層記憶（短期 / 中期 / 長期、ペルソナ別分離、S3 + JSON） |
| 構造化メモリ | DynamoDB（人物・イベント・タスクのエンティティ保持） |
| 後処理 | Lambda + Bedrock（会話分類）＋ Delivery Router（マルチチャネル配信） |
| 永続化 | S3（会話ログ / 記憶 / 予習情報、SSE-S3 暗号化）＋ DynamoDB（ペルソナ / 設定 / 構造化メモリ） |
| 配信 | Slack / LINE / Email / Google Calendar / Notion（ユーザー選択式） |
| 認証 | セッションベース（HttpOnly Cookie）＋ OAuth 2.0（Google / Slack） |
| シークレット | AWS Secrets Manager |
| 配布 | Apple TestFlight（予選）→ 本番展開（決勝） |

**詳細は [Application Design ドキュメント](./aidlc-docs/inception/application-design/application-design.md) を参照。**

---

## 設計ドキュメント（Inception フェーズ成果物）

本プロダクトは AWS の **AI-DLC（AI-Driven Development Lifecycle）** に基づき設計されている。

### 審査対象 4 成果物

| 成果物 | 場所 |
|-------|------|
| 要件定義 | [`requirements.md`](./aidlc-docs/inception/requirements/requirements.md) |
| ペルソナ | [`personas.md`](./aidlc-docs/inception/user-stories/personas.md) |
| ユーザーストーリー | [`stories.md`](./aidlc-docs/inception/user-stories/stories.md) |
| アプリケーション設計 | [`application-design.md`](./aidlc-docs/inception/application-design/application-design.md) |
| Unit of Work 計画 | [`unit-of-work.md`](./aidlc-docs/inception/application-design/unit-of-work.md) |

### 補助ドキュメント

| ドキュメント | 場所 |
|-----------|------|
| 実行計画 | [`execution-plan.md`](./aidlc-docs/inception/plans/execution-plan.md) |
| コンポーネント定義 | [`components.md`](./aidlc-docs/inception/application-design/components.md) |
| メソッドシグネチャ | [`component-methods.md`](./aidlc-docs/inception/application-design/component-methods.md) |
| サービス層 | [`services.md`](./aidlc-docs/inception/application-design/services.md) |
| 依存関係 | [`component-dependency.md`](./aidlc-docs/inception/application-design/component-dependency.md) |
| Unit 依存 | [`unit-of-work-dependency.md`](./aidlc-docs/inception/application-design/unit-of-work-dependency.md) |
| ストーリー⇄Unit マッピング | [`unit-of-work-story-map.md`](./aidlc-docs/inception/application-design/unit-of-work-story-map.md) |
| 矛盾・実現可能性レビュー | [`consistency-feasibility-review.md`](./aidlc-docs/inception/application-design/consistency-feasibility-review.md) |
| 監査ログ | [`audit.md`](./aidlc-docs/audit.md) |
| プロジェクト状態 | [`aidlc-state.md`](./aidlc-docs/aidlc-state.md) |

---

## Unit 分解の思想

**ユーザージャーニー軸 × 3 本柱軸 のハイブリッド** で 4 Unit に分解した。

| Unit | ジャーニー | 主軸の柱 | 含むコンポーネント |
|------|----------|---------|-----------------|
| **U1 入口 Unit** | Pre-call | ⚡ ゼロ UX | Native App Shell, Widget Extension, Call UI |
| **U2 会話 Unit** | In-call | 🫂 受け止める力 | Voice Session Manager, Accepting Engine, Persona Manager, Aizuchi Engine |
| **U3 予習 Unit** | Pre-call + In-call | 🧠 あなたを知っていること | Preview Agent, Data Connectors, Memory Store, Structured Memory |
| **U4 整理・配信 Unit** | Post-call | 🫂 受け止める力（裏側） | Analysis Engine, Delivery Router |

### なぜこう分けたか

1. **各 Unit が単一のジャーニー局面と単一の柱を主に担う** — 説明性が高い
2. **チーム 3 名で並行開発可能な粒度** — ハッカソン期間内の並列作業前提
3. **MVP Must 5 本を最小コンポーネントで動かせる** — 実現可能性重視
4. **審査員への "なぜこう分けたか" が自明** — 書類審査の生命線
5. **各 Unit 間のインターフェースが明確** — 結合度低く保つ

詳細は [`unit-of-work.md`](./aidlc-docs/inception/application-design/unit-of-work.md) を参照。

---

## MVP ストーリー（Must 5 本）

予選デモで必ず動かす、プロダクトの魂を体現する 5 本:

| ID | ストーリー | 担当 Unit |
|----|---------|---------|
| **1.1** | ロック画面から 1 タップで話し始める | U1 |
| **2.1** | AI から先に話しかけてくれる | U1 + U2 + U3 |
| **2.2** | 整理されていない愚痴を、そのまま受け止めてもらう | U2 |
| **2.3** | 黙っていても AI から話題を振る or 相槌だけで寄り添う | U2 + U3 |
| **4.1** | 複数データソースを束ねて、あなたを知る | U3 |

全 15 ストーリーは [`stories.md`](./aidlc-docs/inception/user-stories/stories.md) を参照。

---

## ペルソナ

### プライマリ: 「話して整理する人」

- 28 歳 IT 企画職、リモートワーク中心
- 文字化が苦手。音声で話しながら気持ちや思考を整理するタイプ
- 疲れた夜には整理する気力もエネルギーも残っていない

### ネガティブ: 「書いて整理する人」

- 32 歳コンサルマネージャー、Obsidian で思考構造化
- 自分で整理するのが得意なので gutitto の価値が刺さらない
- → このペルソナを **明示的にターゲット外** とすることで、プロダクトの覚悟を示す

詳細は [`personas.md`](./aidlc-docs/inception/user-stories/personas.md) を参照。

---

## ロードマップ

| フェーズ | タイムライン | 内容 |
|--------|-----------|-----|
| 書類審査 | 〜2026-05-10 | Inception 成果物提出 ← **現在ここ** |
| 予選デモ準備 | 審査合格後 | U1 / U2 / U3 並行開発、iOS TestFlight 配布 |
| 予選プレゼン | 日程未確定 | 動作する MVP ＋ プレゼンテーション |
| 決勝準備 | 予選通過後 | U4 追加、AWS 本格デプロイ、Android 版、Knowledge Graph 本格化、Vector DB 導入 |
| 決勝 | 日程未確定 | AWS 上での実デプロイデモ |

---

## ハッカソン情報

本プロダクトは **AWS Summit Japan 2026 AI-DLC ハッカソン「人をダメにするサービス」** への応募作品。

- 出典: https://pages.awscloud.com/summit-japan-2026-hackathon-reg.html
- 書類審査の対象: `aidlc-docs/` 配下の 4 成果物（要件・ストーリー・設計・Unit of Work）
- AI-DLC 成果物の公開URL は 2026-05-12 正午までに運営に連絡

---

## チーム

**team-gutitto（仮）** 

| 役割 | 担当メンバー | 担当 Unit |
|-----|----------|---------|
| 代表 / UI デザイン / プレゼン / ビジネスデザイン | U1 入口 Unit |
| バックエンド / 設計 / 技術検証 | 未定 1 | U2 会話 Unit |
| インフラ / 技術検証 / 画像編集 | 未定 2 | U3 予習 Unit |

---

<p align="center">
  <b>話して、寝るだけ。整理も体裁も要らない。<br/>ダメになれる存在が、そばにいる。</b>
</p>

<p align="center">
  <i>安心して、何でも話していいよ</i>
</p>

---

## ライセンス

TBD（提出時に確定）
