# Unit of Work — 相撲一番（SUMO ICHIBAN）

## ユニット一覧

| ID | ユニット名 | 技術 | 優先度 | 依存ユニット | 相撲技 |
|----|-----------|------|--------|------------|--------|
| U-01 | frontend | Next.js | 高 | U-02 | 仕切り |
| U-02 | orchestrator | Python 3.12 / Lambda | 高 | U-03, U-04, U-05 | 親方 |
| U-03 | encouragement | Python 3.12 / Lambda | 高 | なし | 決まり手 |
| U-04 | analysis | Python 3.12 / Lambda | 高 | なし | 見合い |
| U-05 | dependency-tracker | Python 3.12 / Lambda | 中 | なし | 番付昇進 |
| U-06 | infra | AWS CDK / TypeScript | 高 | 全ユニット | 土俵築き |

---

## ユニット詳細

### U-01: frontend（仕切り）
- **説明**: Next.js による Web UI 全体
- **主要コンポーネント**: C-01
- **主要機能**:
  - 場面選択画面（仕事シーン / 日常生活シーン）
  - お相撲さん同席モード（助言メッセージ・相撲技名・音声再生）
  - 依存度ダッシュボード（番付表示・仕事/日常シーン別利用比率）
  - 利用履歴画面
- **ディレクトリ**: `frontend/`

### U-02: orchestrator（親方）
- **説明**: セッション管理・各 Lambda の制御
- **主要コンポーネント**: C-04
- **主要機能**:
  - セッション開始/終了 API
  - シーンカテゴリ（仕事/日常）の判別と相撲技の選択
  - 分析結果受信・助言生成トリガー
  - WebSocket 接続管理
- **ディレクトリ**: `backend/orchestrator/`

### U-03: encouragement（決まり手）
- **説明**: 助言コンテンツ生成（Bedrock + Polly）
- **主要コンポーネント**: C-05, C-09, C-11, C-12
- **主要機能**:
  - Bedrock による助言テキスト生成（相撲技・実用的アドバイス）
  - 仕事シーン: 具体的な対処フレーズ・交渉術
  - 日常シーン: 献立提案・買い物リスト・体調アドバイス・スケジュール整理
  - Polly による音声ファイル生成
  - S3 への音声ファイル保存
- **ディレクトリ**: `backend/encouragement/`

### U-04: analysis（見合い）
- **説明**: ユーザーの状況検知（Transcribe）
- **主要コンポーネント**: C-06, C-10
- **主要機能**:
  - Transcribe によるリアルタイム音声認識
  - 緊張ワード検知（仕事シーン）→ 押し出し・寄り切り技発動
  - 迷いワード検知（日常シーン）→ 引き落とし・割り出し技発動
  - 疲労ワード検知（健康シーン）→ 土俵際の踏ん張り技発動
  - 発話速度（単語/秒）の計算による早口・詰まり検知
- **ディレクトリ**: `backend/analysis/`

### U-05: dependency-tracker（番付昇進）
- **説明**: 依存度スコア計算・番付管理
- **主要コンポーネント**: C-07, C-13
- **主要機能**:
  - セッション終了時の依存度スコア計算（仕事/日常シーン別重み付け）
  - DynamoDB への利用履歴・スコア保存
  - 番付更新（序ノ口 → 序二段 → 三段目 → 幕下 → 十両 → 幕内 → 横綱）
  - 番付昇進時の特別メッセージ生成
- **ディレクトリ**: `backend/dependency-tracker/`

### U-06: infra（土俵築き）
- **説明**: AWS CDK によるインフラ全体の IaC
- **主要コンポーネント**: C-14
- **主要リソース**:
  - Lambda 関数（全 5 つ）
  - API Gateway（REST + WebSocket）
  - DynamoDB テーブル（3 つ）
  - S3 バケット
  - IAM ロール・ポリシー
- **ディレクトリ**: `infra/`

---

## モノレポ構成

```
sumo-ichiban/
├── frontend/              # U-01: Next.js Web UI（仕切り）
│   ├── src/
│   │   ├── app/
│   │   ├── components/
│   │   └── lib/
│   └── package.json
├── backend/
│   ├── orchestrator/      # U-02: Orchestrator Lambda（親方）
│   ├── encouragement/     # U-03: Encouragement Lambda（決まり手）
│   ├── analysis/          # U-04: Analysis Lambda（見合い）
│   └── dependency-tracker/ # U-05: Dependency Tracker Lambda（番付昇進）
├── infra/                 # U-06: AWS CDK（土俵築き）
│   ├── lib/
│   └── bin/
└── inception/             # AI-DLC ドキュメント
```

---

## 実装優先順位

1. **U-06 infra（土俵築き）** — CDK スタック骨格（Lambda + API Gateway + DynamoDB + S3）
2. **U-03 encouragement（決まり手）** — Bedrock + Polly の助言生成（コアバリュー）
3. **U-02 orchestrator（親方）** — セッション管理・シーン判別・フロー制御
4. **U-04 analysis（見合い）** — Transcribe 音声解析（緊張/迷い/疲労ワード・沈黙・発話速度）
5. **U-01 frontend（仕切り）** — Web UI（場面選択 + 同席モード + 日常シーン）
6. **U-05 dependency-tracker（番付昇進）** — 番付スコア・デモ映え向上
