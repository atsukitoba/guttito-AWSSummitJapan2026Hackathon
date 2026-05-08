# Requirements Verification Questions

以下の質問に答えてください。各質問の `[Answer]:` タグの後ろに選択肢の文字（A/B/C/...）を記入してください。
もし選択肢に該当するものがなければ最後の「Other」を選び、自由記述してください。

---

## Intent Analysis Summary（AI側の初期理解）

添付の `PRESENTATION.md` と `-DM-.md` から、以下のように理解しました:

- **プロダクト**: gutitto（ぐちっと）
- **キャッチコピー**: 「グッとくる、わかってくれる誰かがいる」
- **コア体験**: スピーカーフォンで気軽に話すだけで、AIキャラクターが「あなたを予習済み」の状態で傾聴し、会話終了後に報連相・タスク・アクションプランを自動生成してSlack等に投稿する
- **ハッカソンテーマ**: 「人をだめにするサービス」
- **プロダクト哲学**: 「だめにする」けれど、その過程で**感情・思考の整理**というアウトカムを提供（＝依存させつつ、依存することで楽に生きられる）
- **主要技術**: Grok Voice API（2025年12月リリース）、Amazon Bedrock、AWS Lambda、S3、Slack/Google Workspace連携、VoIP（Amazon Connect/Chime SDK または独自）
- **MVP範囲**: 通話 → 会話（相槌・自発発話・予習） → 会話分析 → Slack投稿

以下、この理解を確定させるための質問です。

---

## Question 1: ハッカソンテーマ「人をだめにするサービス」の解釈

gutittoが「人をだめにする」とはどういう状態を指しますか？これは要件の根幹なので確認させてください。

A) 思考停止で頼りきり — ユーザーは何も整理せず、ただ話すだけで全て任せる。"考える労力ゼロ"を価値とする
B) 心の依存 — ユーザーがgutittoに感情的に依存し、「このAIがいないと夜を過ごせない」状態を意図的に作る
C) 労働からの逃避 — 仕事の報連相や資料作成を完全代行し、ユーザーが"面倒な作業"から永久に解放される
D) 全部入り（A+B+C） — 思考も感情も労働も、全部AIにだらしなく頼れる状態を提供
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A、感情の整理をする労力もいらない

---

## Question 2: 対象となる1次ユーザーのペルソナ

MVP段階で最も重視するユーザー像はどれですか？（複数選択可：カンマ区切り）

A) 20代〜30代の若手会社員（リモートワーク中心、同期が少なく孤独気味）
B) 30代〜40代のミドル層（家庭と仕事の板挟み、愚痴の吐き出し口がない）
C) 20代〜30代の独身女性（日中業務で気を張り詰め、夜の"切り替え"が欲しい）
D) 営業職全般（定性的な活動報告がサンクコスト化している）
E) フリーランス・個人事業主（孤独に作業し、日報相手がいない）
F) 特定業界（介護、医療、教育など対人消耗の激しい職業）
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A,B
---

## Question 3: MVP（予選デモで動かす範囲）のスコープ

予選段階（5/10書類審査）では **aidlc-docs/** の中身が審査対象ですが、技術的なMVPとしてはどこまで実装する前提で要件を組みますか？

A) 最小コア — 通話 + AI応答 + ペルソナ選択のみ。予習と自動投稿は構想として記載
B) 予習付き — A + カレンダー連携/Slack読み取りによる"予習"機能
C) フルループ — B + 会話終了後のBedrock分析 + Slack自動投稿まで完結
D) フル＋α — C + ダッシュボード or RAG or メンタル分析 のいずれか1つ追加
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## Question 4: 通話インターフェースの実装方式

ユーザーが通話を開始する入口はどう設計しますか？

A) LINE通話連携 — LINE Messaging APIとVoIPの組み合わせ。最もゼロUX
B) Amazon Connect/Chime SDK — 電話番号を発行してPSTN接続
C) Web App — ブラウザでボタンタップ → WebSocket 通話（HTML/JS フロント）
D) iOS/Androidネイティブアプリ — PWAまたはネイティブで通話体験
X) Other (please describe after [Answer]: tag below)

[Answer]: 
AまたはD
ゼロUXを徹底。特定のアプリを開いてログインではなく、誰かに連絡をとる感覚で
---

## Question 5: 「あなたを予習する」の情報ソース

MVPで「予習」に使うデータソースはどれですか？（複数選択可：カンマ区切り）

A) Googleカレンダー — 当日/翌日の予定
B) Slack — 直近のチャンネル/DMメッセージ
C) Gmail/Google Workspace — 未読メール、重要メール
D) 過去の会話履歴 — gutitto自身の過去セッションのサマリ
E) ウェアラブル（睡眠/心拍/歩数） — Apple Health, Google Fit, Fitbit
F) 位置情報 — 自宅/会社/外出先の判定
G) 天気/ニュース — 文脈の豊富化
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A　B　C  D
---

## Question 6: ペルソナ（AIキャラクター）のMVP範囲

デモ・書類審査の時点でどれくらいのペルソナを用意しますか？

A) 1体固定 — 「まなみ」のみ、切替機能なし
B) 2〜3体 — 彼女系／頼れる上司系／おばちゃん系 など代表的な3体
C) 5体以上 — 彼女・旦那・女上司・イケオジ・おばちゃんの5体セット
D) カスタマイズUI — ユーザーが自由に作れるUI込み
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
---

## Question 7: 会話後のアウトプット先

通話終了後に自動生成されたアウトプットはどこに送りますか？

A) Slack一本 — 日報チャンネル + 個人DMの2本立て
B) Slack + メール — 幅広くフォロー
C) Slack + Googleカレンダー/ToDoリスト追加
D) Slack + LINE + Notion など複数チャネル展開
E) アプリ内ダッシュボードのみ — 外部投稿は次フェーズ
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question 8: 会話内容の分類軸

Bedrockで会話を分析する際、どの軸で分類・整理しますか？

A) 仕事／プライベート の2分類
B) 報告／連絡／相談 の3分類（報連相）
C) 感情／タスク／願望（やりたいこと） の3分類
D) 複合 — 仕事/プライベート × 報告/連絡/相談/タスク/願望 のマトリクス
X) Other (please describe after [Answer]: tag below)

[Answer]: 
D
---

## Question 9: プライバシー・データ保護

「本音の一次情報」を扱うため、プライバシーは重要です。MVPではどこまで対応しますか？

A) 最小限 — 会話ログはS3にユーザーID付きで保存、暗号化なし
B) 標準 — S3暗号化、個人データは個別バケット、TTL設定
C) 堅牢 — S3暗号化 + KMS + ユーザーによる削除UI + データエクスポート権
D) 最堅牢 — C + オンプレ暗号化 + 完全ユーザーコントロール（ローカル保存オプション）
X) Other (please describe after [Answer]: tag below)

[Answer]: 

---

## Question 10: 決勝進出時のAWSアーキテクチャ方針

決勝進出時にAWS上にデプロイする前提で、基本方針はどちらですか？

A) フルサーバーレス — Lambda + API Gateway WebSocket + DynamoDB + S3 + Bedrock（スケール重視）
B) ハイブリッド — EC2/ECS（音声WebSocketプロキシ担当）＋ Lambda（非同期処理）＋ Bedrock
C) コンテナ中心 — ECS Fargate + ALB + RDS/DynamoDB + Bedrock
D) 決勝時に判断 — MVPはシンプルに作り、決勝向けに再アーキテクト
X) Other (please describe after [Answer]: tag below)

[Answer]: 
A
---

## Question 11: 成功指標（KPI）

このサービスの成功を何で測りますか？（複数選択可：カンマ区切り）

A) 通話時間の中央値 — 長く話してもらえるほど「依存」成功
B) 継続率（Weekly Active） — 毎日/毎週使ってもらえる
C) 会話後のSlack投稿承認率 — AIが生成した日報が実際に使われる割合
D) 「話してスッキリした」フィードバック率 — 感情面での効果
E) タスク完了率 — AI提案のアクションが実行される割合
F) ネットプロモータースコア（NPS） — 人に勧めたくなるか
X) Other (please describe after [Answer]: tag below)

[Answer]: 
日報投稿機能は除いて
B
---

## Question 12: 「人をだめにする」の倫理的バランス

「依存させる」ことが価値である一方、ユーザーの自律を損なうリスクもあります。どう扱いますか？

A) 割り切る — 倫理議論は回避し、「楽になる＝正義」で設計
B) 自律促進機能を埋め込む — 会話後アウトプットで「自分で決める次の一歩」を必ず含める
C) 依存度モニタリング — 過度な利用時にAI側から「たまには友達に話そう？」と促す
D) B + C 両方
X) Other (please describe after [Answer]: tag below)

[Answer]: 
B
ぐちっとと最中はとにかく吐き出すため、自立は促さないが、終わったあとのフィードバックは積み上げや変革をねらう
---

## Question 13: Grok Voice API への依存リスク

Grok Voice API（xAI）はサービスの中核ですが、依存リスクもあります。どう扱いますか？

A) 全面採用 — Grokのみ。リリース間もなく差別化要素として活用
B) Grokメイン + フォールバック — Amazon Polly/Transcribe で代替可能な設計に
C) マルチプロバイダ — Grok / OpenAI Realtime / Google Gemini Live を切替可能に
D) 決勝時のみAWS中心に再構成 — MVPはGrok、決勝はBedrock + Polly/Transcribe
X) Other (please describe after [Answer]: tag below)

[Answer]: 
C
grok
---

## Question 14: パフォーマンス要件（非機能要件）

音声応答の遅延はどれくらいを許容しますか？（予選デモ時の目標）

A) 超低遅延 — 500ms以内（電話体験を損なわない）
B) 低遅延 — 1秒以内（体感的に違和感なし）
C) 中程度 — 2秒以内（少し待つが許容）
D) ベストエフォート — Grok API次第で、特に目標値なし
X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 15: 書類審査で審査員に刺さる"Unit分解の観点"

審査員は「なぜこう分けたか」を重視します。Unit分解（作業分割）の軸として何を重視しますか？

A) **ユーザージャーニー軸** — 予習／会話体験／分析・出力 の3軸でUnitを切る
B) **責務軸（データフロー）** — 入力層／コア処理／出力層 のレイヤー分割
C) **並行開発しやすさ軸** — チーム3人が独立して作業できる粒度（フロント/バック/AI統合）
D) **ビジネス価値軸** — MVPコア／予習機能／自動アウトプット／分析ダッシュボード のように価値別
E) **AWSサービス境界軸** — Connect/Chime、Lambda群、Bedrock、S3 のようにクラウドサービス単位
X) Other (please describe after [Answer]: tag below)

[Answer]: 

---

## Extension Opt-In Questions

AI-DLCには「エクステンション」という追加ルールセットがあります。ハッカソン審査品質に影響する可能性があるため、個別に判断してください。

### Question 16: Security Baseline 拡張
セキュリティのベースラインルール（認証・認可・Secrets管理・入力検証・HTTPS強制など）を必須制約として適用しますか？

A) Yes — 全てのSECURITYルールをブロッキング制約として適用（本番運用想定プロジェクトに推奨）
B) No — 全てのSECURITYルールをスキップ（PoC・プロトタイプ・実験プロジェクトに適切）
X) Other (please describe after [Answer]: tag below)

[Answer]: B

### Question 17: Property-Based Testing 拡張
プロパティベーステスト（PBT）のルールを必須制約として適用しますか？

A) Yes — 全てのPBTルールをブロッキング制約として適用（ビジネスロジック/データ変換/シリアライゼーション/状態コンポーネントを含むプロジェクトに推奨）
B) Partial — 純粋関数とシリアライゼーションラウンドトリップのみPBTルールを適用（限定的なアルゴリズム複雑度のプロジェクトに適切）
C) No — 全てのPBTルールをスキップ（単純なCRUDアプリ、UI中心のプロジェクト、薄い統合層で複雑なビジネスロジックがない場合に適切）
X) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## 回答後の確認

全て回答したら、「回答完了」または「done」とお伝えください。回答内容を分析し、曖昧な箇所があれば追加の質問を生成します。曖昧さがなければ `requirements.md` を生成します。
