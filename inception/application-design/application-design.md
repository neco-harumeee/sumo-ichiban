# アプリケーション設計 — TRAINING LOCK

## 設計サマリー

| 項目 | 決定内容 |
|------|---------|
| **フロントエンド** | Next.js（React + SSR） |
| **バックエンド** | 機能別 Lambda（Orchestrator + Social Shield + Motivation + Logistics + WS Notifier） |
| **AI エンジン** | Amazon Bedrock Agents（Claude 3 Sonnet / Haiku） |
| **状態管理** | Amazon DynamoDB（4 テーブル） |
| **通信** | REST（API Gateway）+ WebSocket（API Gateway WebSocket） |
| **音声生成** | Amazon Polly → S3 |
| **インフラ管理** | AWS CDK（TypeScript） |

---

## アーキテクチャ概要

```
+--------------------------------------------------+
|  FRONTEND                                        |
|  Next.js (Vercel / S3+CloudFront)                |
|  - ダッシュボード                                 |
|  - トレーニング宣言フォーム                        |
|  - 設定画面                                      |
|  - リアルタイム通知表示                            |
+--------------------------------------------------+
         |  REST                  | WebSocket
         v                        v
+----------------+    +------------------------+
| API Gateway    |    | API Gateway WebSocket  |
| REST API       |    |                        |
+----------------+    +------------------------+
         |                        |
         v                        v
+----------------+    +------------------------+
| Orchestrator   |    | WS Notifier Lambda     |
| Lambda         |    | (push updates)         |
+----------------+    +------------------------+
    |      |      |         ^    ^    ^
    |      |      |         |    |    |
    v      v      v         |    |    |
+-------+ +----------+ +----------+
|Social | |Motivation| |Logistics |
|Shield | |Rebuttal  | |Lambda    |
|Lambda | |Lambda    | +----------+
+-------+ +----------+
    |           |
    v           v
+----------------------------------+
| Amazon Bedrock Agents            |
| (Claude 3 Sonnet / Haiku)        |
| - social-shield-actions          |
| - motivation-actions             |
| - logistics-actions              |
+----------------------------------+
                |
    +-----------+-----------+
    |           |           |
    v           v           v
+-------+  +-------+  +--------+
| Polly |  |  SES  |  | PA-API |
+-------+  | Mock  |  | Mock   |
    |      +-------+  +--------+
    v
  +----+
  | S3 |
  +----+

  全 Lambda
      |
      v
+----------+
| DynamoDB |
| Sessions |
| Settings |
| Orders   |
| WS Conns |
+----------+

  インフラ全体
      |
      v
+----------+
| AWS CDK  |
|(TypeScript)|
+----------+
```

---

## コンポーネント一覧（参照）

詳細は `components.md` を参照。

| ID | コンポーネント | 技術 |
|----|--------------|------|
| C-01 | Web UI | Next.js |
| C-02 | API Gateway REST | Amazon API Gateway |
| C-03 | API Gateway WebSocket | Amazon API Gateway |
| C-04 | Lambda Orchestrator | Python 3.12 |
| C-05 | Lambda Social Shield | Python 3.12 |
| C-06 | Lambda Motivation Rebuttal | Python 3.12 |
| C-07 | Lambda Logistics | Python 3.12 |
| C-08 | Lambda WS Notifier | Python 3.12 |
| C-09 | Bedrock Agent | Claude 3 Sonnet/Haiku |
| C-10 | Amazon Polly | — |
| C-11 | Amazon S3 | — |
| C-12 | Amazon DynamoDB | — |
| C-13 | AWS CDK | TypeScript |

---

## サービス一覧（参照）

詳細は `services.md` を参照。

| ID | サービス | 主要フロー |
|----|---------|-----------|
| S-01 | TrainingSessionService | セッション開始/終了/状態管理 |
| S-02 | SocialShieldService | メール返信・家族連絡の自動生成・送信 |
| S-03 | MotivationRebuttalService | サボり検知・反論生成・音声送信 |
| S-04 | LogisticsService | ギア発注判断・PA-API 連携 |
| S-05 | RealtimeNotificationService | WebSocket リアルタイム通知 |

---

## 主要設計決定と根拠

| 決定 | 根拠 |
|------|------|
| 機能別 Lambda 分割 | 各機能が独立してスケール・デプロイ可能。ハッカソンでの部分的なデモも容易 |
| Bedrock Agents 採用 | アクション自動判断により「AI が考えて動く」デモ映えが向上 |
| REST + WebSocket 併用 | 操作は REST でシンプルに、状態更新は WebSocket でリアルタイム表示 |
| DynamoDB 永続化 | デモ中にページリロードしても状態が保持される |
| AWS CDK | インフラをコードで管理し、ハッカソン後の再現・共有が容易 |
| モック実装方針 | GitHub 公開コードには実際の送信処理を含めず、デモ動画で実動作を示す |
