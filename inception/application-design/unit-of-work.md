# ユニット・オブ・ワーク定義 — TRAINING LOCK

## 分解方針

| 項目 | 決定内容 |
|------|---------|
| **分割粒度** | 機能別 Lambda ごとに 1 ユニット |
| **リポジトリ構成** | モノレポ（frontend / backend / infra） |
| **共通コード管理** | 各 Lambda にコピー（シンプル、依存なし） |
| **CDK スタック** | 1 スタックにすべてのリソースをまとめる |

---

## ユニット一覧

### Unit 1: Frontend（Web UI）
| 項目 | 内容 |
|------|------|
| **名称** | frontend |
| **技術** | Next.js（React + SSR） |
| **責務** | トレーニング宣言 UI / 設定画面 / リアルタイム状態表示 / アクション履歴 |
| **対応コンポーネント** | C-01 Web UI |
| **対応サービス** | S-01（セッション開始/終了操作）/ S-05（WebSocket 受信） |
| **ディレクトリ** | `frontend/` |

### Unit 2: Lambda — Orchestrator
| 項目 | 内容 |
|------|------|
| **名称** | backend/orchestrator |
| **技術** | Python 3.12 / AWS Lambda |
| **責務** | セッション管理・各機能 Lambda の起動制御・DynamoDB 状態管理 |
| **対応コンポーネント** | C-04 Lambda Orchestrator |
| **対応サービス** | S-01 TrainingSessionService |
| **ディレクトリ** | `backend/orchestrator/` |

### Unit 3: Lambda — Social Shield（社会的シールド）
| 項目 | 内容 |
|------|------|
| **名称** | backend/social-shield |
| **技術** | Python 3.12 / AWS Lambda |
| **責務** | Bedrock Agent 呼び出し・メール返信文生成・家族連絡文生成・SES モック送信 |
| **対応コンポーネント** | C-05 Lambda Social Shield |
| **対応サービス** | S-02 SocialShieldService |
| **ディレクトリ** | `backend/social-shield/` |

### Unit 4: Lambda — Motivation Rebuttal（モチベ反論）
| 項目 | 内容 |
|------|------|
| **名称** | backend/motivation-rebuttal |
| **技術** | Python 3.12 / AWS Lambda / Amazon Polly / S3 |
| **責務** | サボり検知・Bedrock Agent で反論生成・Polly 音声合成・S3 保存・メール/LINE 送信 |
| **対応コンポーネント** | C-06 Lambda Motivation Rebuttal |
| **対応サービス** | S-03 MotivationRebuttalService |
| **ディレクトリ** | `backend/motivation-rebuttal/` |

### Unit 5: Lambda — Logistics（物流連携）
| 項目 | 内容 |
|------|------|
| **名称** | backend/logistics |
| **技術** | Python 3.12 / AWS Lambda |
| **責務** | Bedrock Agent による発注判断・PA-API モック呼び出し・発注通知送信 |
| **対応コンポーネント** | C-07 Lambda Logistics |
| **対応サービス** | S-04 LogisticsService |
| **ディレクトリ** | `backend/logistics/` |

### Unit 6: Lambda — WebSocket Notifier
| 項目 | 内容 |
|------|------|
| **名称** | backend/ws-notifier |
| **技術** | Python 3.12 / AWS Lambda |
| **責務** | WebSocket 接続管理・リアルタイム通知プッシュ |
| **対応コンポーネント** | C-08 Lambda WS Notifier |
| **対応サービス** | S-05 RealtimeNotificationService |
| **ディレクトリ** | `backend/ws-notifier/` |

### Unit 7: Infrastructure（CDK）
| 項目 | 内容 |
|------|------|
| **名称** | infra |
| **技術** | AWS CDK（TypeScript） |
| **責務** | 全 AWS リソース定義（API Gateway / Lambda / DynamoDB / S3 / Bedrock / Polly） |
| **対応コンポーネント** | C-13 AWS CDK |
| **スタック** | TrainingLockStack（1 スタック） |
| **ディレクトリ** | `infra/` |

---

## モノレポ ディレクトリ構成

```
training-lock/                    # リポジトリルート
├── frontend/                     # Unit 1: Next.js Web UI
│   ├── src/
│   │   ├── app/                  # Next.js App Router
│   │   │   ├── page.tsx          # ダッシュボード
│   │   │   ├── settings/
│   │   │   └── history/
│   │   ├── components/           # React コンポーネント
│   │   └── lib/                  # API クライアント / WebSocket
│   ├── public/
│   ├── package.json
│   └── next.config.js
│
├── backend/                      # バックエンド Lambda 群
│   ├── orchestrator/             # Unit 2
│   │   ├── handler.py
│   │   ├── dynamodb_client.py    # 共通コードコピー
│   │   ├── bedrock_client.py     # 共通コードコピー
│   │   └── requirements.txt
│   ├── social-shield/            # Unit 3
│   │   ├── handler.py
│   │   ├── dynamodb_client.py    # 共通コードコピー
│   │   ├── bedrock_client.py     # 共通コードコピー
│   │   ├── ses_mock.py           # SES モック
│   │   └── requirements.txt
│   ├── motivation-rebuttal/      # Unit 4
│   │   ├── handler.py
│   │   ├── dynamodb_client.py    # 共通コードコピー
│   │   ├── bedrock_client.py     # 共通コードコピー
│   │   ├── polly_client.py
│   │   ├── s3_client.py
│   │   └── requirements.txt
│   ├── logistics/                # Unit 5
│   │   ├── handler.py
│   │   ├── dynamodb_client.py    # 共通コードコピー
│   │   ├── bedrock_client.py     # 共通コードコピー
│   │   ├── pa_api_mock.py        # PA-API モック
│   │   └── requirements.txt
│   └── ws-notifier/              # Unit 6
│       ├── handler.py
│       ├── dynamodb_client.py    # 共通コードコピー
│       └── requirements.txt
│
├── infra/                        # Unit 7: AWS CDK
│   ├── bin/
│   │   └── training-lock.ts
│   ├── lib/
│   │   └── training-lock-stack.ts
│   ├── package.json
│   └── cdk.json
│
├── docs/                         # ドキュメント（aidlc-docs 以外）
└── README.md
```
