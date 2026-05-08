# Consistency & Feasibility Review

全成果物（requirements.md / stories.md / personas.md / application-design 各種）を横断レビューした結果を記録する。

---

## 🚨 重大度: High（矛盾 or 実現困難）

### R1: PWA と「ロック画面ウィジェット」の根本的矛盾 ★最重要
**場所**: requirements.md R-Entry-1 (a)(b)(c), R-Entry-2 / stories.md Story 1.1, 1.2 / application-design.md

**問題**:
- 現状の設計: **PWA** として実装
- しかし、**iOS のロック画面ウィジェット** は WidgetKit（Swift / SwiftUI）でしか実装できない
- **Androidのロック画面ウィジェット** も AppWidgetProvider（ネイティブ）が必要
- **Siri / Google Assistant ショートカット** も App Intents / Android App Actions でネイティブ前提
- **Face ID / Touch ID のバックグラウンド認証** も PWA では不可（WebAuthnで類似体験は可能だが制約あり）

**影響**:
- R-Entry-1 の (a)(b)(c)(e) が PWA では実現不能
- ゼロUX柱の主張が技術的に成立しない
- ハッカソン予選デモで「ロック画面1タップ」が動かない可能性大

**質問 Q1**: この矛盾をどう解決しますか？ ※後述

---

### R2: 500ms 応答遅延目標の現実性
**場所**: requirements.md NFR-Performance-1, R-Voice-1 / stories.md 1.1 シナリオ1-2

**問題**:
- 目標: ユーザー発話終了 → AI応答音声再生開始まで **500ms 以内**
- 実測感: Grok Voice API の実レイテンシは **1〜2秒** 程度（VAD検出＋LLM推論＋音声生成＋ネットワーク往復）
- さらに間に AWS Lambda / API Gateway / WebSocket プロキシが入ると追加で 100〜300ms
- 500ms は**業界最先端レベル**（GPT-4o Realtime でも600〜800msクラス）

**影響**:
- 達成できない KPI を掲げると審査でマイナス評価
- NFR は実現可能な数値にすべき

**質問 Q2**: 500ms は緩和すべき。どこに設定しますか？

---

### R3: Siri ショートカット × PWA の実現不可
**場所**: requirements.md R-Entry-1 (e), stories.md Story 1.1 シナリオ4・5

**問題**:
- R1 と同じルーツ。PWA では Siri カスタムショートカットを登録できない
- iOS Shortcuts アプリでURL起動は可能だが「Hey Siri、まなみ」のような応答はネイティブ必須

**影響**:
- 「声で呼び出す」体験が PWA では成立しない

**質問 Q3**: Siri 要件はどうしますか？

---

## ⚠️ 重大度: Medium（設計複雑度・コスト懸念）

### R4: Knowledge Graph の MVP 必要性
**場所**: application-design.md component, stories.md 4.1

**問題**:
- Knowledge Graph を DynamoDB 隣接リスト形式で実装する設計
- ただし MVP の Must 5本を動かす範囲では、**Bedrock のコンテキストウィンドウにプロンプトを詰めるだけ**で十分な可能性
- グラフDB設計・エンティティ抽出・関係性推論はかなりの工数

**影響**:
- 予選期間（実質数日）で KG の実装は困難
- KG なしで MVP Must 5本を成立させられるなら、簡略化すべき

**質問 Q4**: MVP段階の Knowledge Graph の扱いは？

---

### R5: OpenSearch Serverless のコスト
**場所**: application-design.md, component-methods.md

**問題**:
- OpenSearch Serverless は最低 $0.24/OCU/h × 2 OCU 常時起動 = **約 $350/月** からスタート
- 予選段階の学習デモで月 $350 は割に合わない
- 代替: pgvector（PostgreSQL）、ChromaDB、FAISS（ローカル）、Pinecone Starter Free

**影響**:
- 予選期間のコスト負担が増大

**質問 Q5**: Vector DB の選定方針は？

---

### R6: 予習エージェントの定期実行 vs 通話前トリガー
**場所**: application-design.md services.md

**問題**:
- 現設計: 「1時間ごと定期実行 or 通話前トリガー」と両対応を記載
- 1時間ごとに全ユーザーの予習を走らせるとコスト高・スケール負荷大
- 通話前トリガーで十分な可能性大（ユーザーが通話しない限り予習不要）

**影響**:
- コスト・複雑度の不必要な増加

**質問 Q6**: 予習エージェントの実行トリガーは？

---

### R7: Memory Store の恒久保存とコスト
**場所**: requirements.md R-Security-1, application-design.md

**問題**:
- 現設計: 短期 TTL 90日、**長期記憶は恒久保存**
- ユーザー増加＋会話蓄積で S3 / Vector DB のコストがリニアに増加
- 実際には長期記憶は「エピソード要約 + 重要人物メタデータ」で十分で、**全文保存は不要**

**影響**:
- 長期運用でのコスト爆発リスク

**質問 Q7**: 長期記憶のスキーマを「要約のみ」に絞るか？

---

### R8: Accepting Engine の品質保証
**場所**: application-design.md, component-methods.md

**問題**:
- 「validateResponse で構造化要求を検出・抑制」とあるが、**Grok の応答を後からチェックするには既に発話済み**
- 事前のプロンプトエンジニアリングのみで防ぐしかない
- 「品質モニタリング」は実時間抑制ではなくログ記録程度になる

**影響**:
- 設計意図が曖昧

**質問 Q8**: 事前プロンプトのみに方針統一するか、それとも後処理で再生成するか？

---

## 💡 重大度: Low（改善提案）

### R9: Requirements ターゲットと Personas の乖離
**場所**: requirements.md ターゲットユーザー vs personas.md

**問題**:
- Requirements: 「20代〜30代若手 + 30代〜40代ミドル」（年齢・職業軸）
- Personas: 「話して整理する人 vs 書いて整理する人」（整理スタイル軸）
- 後者の方が鋭いので requirements.md を更新すべき

**修正**: 自動反映可能

---

### R10: 相槌タイミング 1500ms のサーバーサイド検出
**場所**: requirements.md R-Voice-3 / stories.md 2.3

**問題**:
- サーバーサイドVADで 1500ms 沈黙検出すると追加レイテンシ発生
- クライアント側で検出→サーバーに通知する設計が自然

**修正**: Application Design で明確化可能

---

### R11: ペルソナ切替時のコンテキスト継承
**場所**: stories.md 2.6

**問題**:
- 「まなみ」との過去会話を「げんぞう」が参照するか不明
- 「前の彼氏のことは覚えてない彼氏」的な矛盾

**質問**（軽め）: Q9 として提示

---

### R12: Slack限定前提のセキュリティ懸念
**場所**: stories.md 3.2

**問題**:
- 会社Slack への自動投稿は情シス規程に抵触する可能性（個人の本音を組織に流す）
- 個人用Slack workspaceを前提にすべき

**修正**: MVP範囲で明示

---

### R13: Google OAuth readonly スコープの同意UI
**場所**: stories.md 4.1

**問題**:
- Calendar + Gmail readonly は「機密データ閲覧」扱いで Google の同意画面が重い
- 「ゼロUX」と矛盾する初期セットアップ体験

**修正**: 初回セットアップ時のみの許容として注釈を追加

---

## 🎯 質問ファイル

上記の重要論点 R1〜R8 を解決する質問を `consistency-questions.md` に作成しました。
修正作業は質問への回答後に実施します。
