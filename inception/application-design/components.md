# コンポーネント定義 — TRAINING LOCK

## コンポーネント一覧

### C-01: Web UI（Next.js）
| 項目 | 内容 |
|------|------|
| **技術** | Next.js（React + SSR） |
| **責務** | トレーニング宣言 UI / 設定画面 / セッション状態表示 / アクション履歴表示 |
| **インターフェース** | REST API（API Gateway）/ WebSocket（API Gateway WebSocket） |
| **主要画面** | ダッシュボード / トレーニング宣言フォーム / 設定画面 / 履歴画面 |

### C-02: API Gateway（REST）
| 項目 | 内容 |
|------|------|
| **技術** | Amazon API Gateway（REST API） |
| **責務** | Web UI からの HTTP リクエストを受け付け、Lambda へルーティング |
| **エンドポイント** | POST /session/start, POST /session/end, GET /session/{id}, PUT /settings |

### C-03: API Gateway（WebSocket）
| 項目 | 内容 |
|------|------|
| **技術** | Amazon API Gateway（WebSocket API） |
| **責務** | セッション中のリアルタイム状態更新を Web UI へプッシュ |
| **イベント** | $connect, $disconnect, sendUpdate |

### C-04: Lambda — Orchestrator
| 項目 | 内容 |
|------|------|
| **技術** | AWS Lambda（Python 3.12） |
| **責務** | セッション開始/終了の受付、DynamoDB への状態保存、各機能 Lambda の起動制御 |
| **トリガー** | API Gateway REST |

### C-05: Lambda — Social Shield（社会的シールド）
| 項目 | 内容 |
|------|------|
| **技術** | AWS Lambda（Python 3.12） |
| **責務** | Bedrock Agent を呼び出してメール返信文・家族連絡文を生成、SES（モック）で送信 |
| **トリガー** | Orchestrator Lambda から直接呼び出し / EventBridge Scheduler |

### C-06: Lambda — Motivation Rebuttal（モチベ反論）
| 項目 | 内容 |
|------|------|
| **技術** | AWS Lambda（Python 3.12） |
| **責務** | サボり検知、Bedrock Agent で反論テキスト生成、Polly で音声化、S3 保存、メール/LINE 送信 |
| **トリガー** | EventBridge Scheduler（終了予定時刻 + X 分後） |

### C-07: Lambda — Logistics（物流連携）
| 項目 | 内容 |
|------|------|
| **技術** | AWS Lambda（Python 3.12） |
| **責務** | Bedrock Agent の判断に基づき PA-API（モック）で商品検索・発注、発注通知送信 |
| **トリガー** | Orchestrator Lambda から呼び出し |

### C-08: Lambda — WebSocket Notifier
| 項目 | 内容 |
|------|------|
| **技術** | AWS Lambda（Python 3.12） |
| **責務** | 各機能 Lambda の実行結果を WebSocket 経由で Web UI へリアルタイム通知 |
| **トリガー** | 各機能 Lambda から呼び出し |

### C-09: Bedrock Agent
| 項目 | 内容 |
|------|------|
| **技術** | Amazon Bedrock Agents（Claude 3 Sonnet / Haiku） |
| **責務** | 状況に応じたテキスト生成（返信文 / 家族連絡 / 反論 / 発注判断）、アクション自動判断 |
| **Action Groups** | social-shield-actions, motivation-actions, logistics-actions |

### C-10: Amazon Polly
| 項目 | 内容 |
|------|------|
| **技術** | Amazon Polly |
| **責務** | 反論テキストを MP3 音声ファイルに変換 |
| **出力** | MP3 ファイル → S3 へ保存 |

### C-11: Amazon S3
| 項目 | 内容 |
|------|------|
| **技術** | Amazon S3 |
| **責務** | Polly 生成音声ファイルの一時保存、署名付き URL の発行 |
| **バケット** | training-lock-audio-{env} |

### C-12: Amazon DynamoDB
| 項目 | 内容 |
|------|------|
| **技術** | Amazon DynamoDB |
| **責務** | トレーニングセッション状態・ユーザー設定・発注履歴・WebSocket 接続 ID の永続化 |
| **テーブル** | Sessions, UserSettings, OrderHistory, WebSocketConnections |

### C-13: AWS CDK（インフラ）
| 項目 | 内容 |
|------|------|
| **技術** | AWS CDK（TypeScript） |
| **責務** | 全 AWS リソースの定義・デプロイ管理 |
| **スタック** | TrainingLockStack |
