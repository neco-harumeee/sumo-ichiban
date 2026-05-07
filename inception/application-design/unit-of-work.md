# Unit of Work — お相撲さんと一緒

## ユニット一覧

| ID | ユニット名 | 技術 | 優先度 | 依存ユニット |
|----|-----------|------|--------|------------|
| U-01 | frontend | Next.js | 高 | U-02 |
| U-02 | orchestrator | Python 3.12 / Lambda | 高 | U-03, U-04, U-05 |
| U-03 | encouragement | Python 3.12 / Lambda | 高 | なし |
| U-04 | analysis | Python 3.12 / Lambda | 高 | なし |
| U-05 | dependency-tracker | Python 3.12 / Lambda | 中 | なし |
| U-06 | infra | AWS CDK / TypeScript | 高 | 全ユニット |

---

## ユニット詳細

### U-01: frontend
- **説明**: Next.js による Web UI 全体
- **主要コンポーネント**: C-01
- **主要機能**:
  - 場面選択画面
  - お相撲さん同席モード（励ましメッセージ・音声再生）
  - 依存度ダッシュボード
  - 利用履歴画面
- **ディレクトリ**: `frontend/`

### U-02: orchestrator
- **説明**: セッション管理・各 Lambda の制御
- **主要コンポーネント**: C-04
- **主要機能**:
  - セッション開始/終了 API
  - 分析結果受信・励まし生成トリガー
  - WebSocket 接続管理
- **ディレクトリ**: `backend/orchestrator/`

### U-03: encouragement
- **説明**: 励ましコンテンツ生成（Bedrock + Polly）
- **主要コンポーネント**: C-05, C-09, C-12, C-13
- **主要機能**:
  - Bedrock による励ましテキスト生成
  - Polly による音声ファイル生成
  - S3 への音声ファイル保存
- **ディレクトリ**: `backend/encouragement/`

### U-04: analysis
- **説明**: ユーザーの緊張状態検知（Transcribe）
- **主要コンポーネント**: C-06, C-10
- **主要機能**:
  - Transcribe によるリアルタイム音声認識・緊張ワード検知・沈黙検知
  - 発話速度（単語/秒）の計算による早口・詰まり検知
- **ディレクトリ**: `backend/analysis/`

### U-05: dependency-tracker
- **説明**: 依存度スコア計算・記録
- **主要コンポーネント**: C-07, C-14
- **主要機能**:
  - セッション終了時の依存度スコア計算
  - DynamoDB への利用履歴・スコア保存
  - 依存度ランク更新
- **ディレクトリ**: `backend/dependency-tracker/`

### U-06: infra
- **説明**: AWS CDK によるインフラ全体の IaC
- **主要コンポーネント**: C-15
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
sumo-with-me/
├── frontend/              # U-01: Next.js Web UI
│   ├── src/
│   │   ├── app/
│   │   ├── components/
│   │   └── lib/
│   └── package.json
├── backend/
│   ├── orchestrator/      # U-02: Orchestrator Lambda
│   ├── encouragement/     # U-03: Encouragement Lambda
│   ├── analysis/          # U-04: Analysis Lambda
│   └── dependency-tracker/ # U-05: Dependency Tracker Lambda
├── infra/                 # U-06: AWS CDK
│   ├── lib/
│   └── bin/
└── aidlc-docs/            # AI-DLC ドキュメント
```

---

## 実装優先順位

1. **U-06 infra** — CDK スタック骨格（Lambda + API Gateway + DynamoDB + S3）
2. **U-03 encouragement** — Bedrock + Polly の励まし生成（コアバリュー）
3. **U-02 orchestrator** — セッション管理・フロー制御
4. **U-04 analysis** — Transcribe 音声解析（緊張ワード・沈黙・発話速度）
5. **U-01 frontend** — Web UI（場面選択 + 同席モード）
6. **U-05 dependency-tracker** — 依存度スコア（デモ映え向上）
