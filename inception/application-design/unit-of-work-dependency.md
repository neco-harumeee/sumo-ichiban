# ユニット依存関係 — TRAINING LOCK

## 依存関係マトリクス

| 依存元 \ 依存先 | U1 Frontend | U2 Orchestrator | U3 Social Shield | U4 Motivation | U5 Logistics | U6 WS Notifier | U7 Infra |
|---------------|-------------|-----------------|------------------|---------------|--------------|----------------|----------|
| **U1 Frontend** | — | ✅ REST | — | — | — | ✅ WS | ✅ デプロイ先 |
| **U2 Orchestrator** | — | — | ✅ 非同期呼び出し | ✅ 非同期呼び出し | ✅ 非同期呼び出し | ✅ 呼び出し | ✅ リソース定義 |
| **U3 Social Shield** | — | — | — | — | — | ✅ 呼び出し | ✅ リソース定義 |
| **U4 Motivation** | — | — | — | — | — | ✅ 呼び出し | ✅ リソース定義 |
| **U5 Logistics** | — | — | — | — | — | ✅ 呼び出し | ✅ リソース定義 |
| **U6 WS Notifier** | — | — | — | — | — | — | ✅ リソース定義 |
| **U7 Infra** | — | — | — | — | — | — | — |

## 開発順序（推奨）

```
Phase 1（並行可能）:
  U7 Infra ──────────────────────────────→ 全リソース定義
  U1 Frontend ───────────────────────────→ UI 実装（API モックで開発）

Phase 2（U7 完了後、並行可能）:
  U2 Orchestrator ───────────────────────→ セッション管理
  U6 WS Notifier ────────────────────────→ WebSocket 通知

Phase 3（U2 完了後、並行可能）:
  U3 Social Shield ──────────────────────→ メール返信・家族連絡
  U4 Motivation Rebuttal ────────────────→ 反論生成・Polly 音声
  U5 Logistics ──────────────────────────→ 物流連携

Phase 4（全ユニット完了後）:
  統合テスト・デモ動画撮影
```

## 共有 AWS リソース（全ユニットが依存）

| リソース | 用途 | 管理ユニット |
|---------|------|------------|
| DynamoDB Sessions | セッション状態 | U7 Infra |
| DynamoDB UserSettings | ユーザー設定 | U7 Infra |
| DynamoDB OrderHistory | 発注履歴 | U7 Infra |
| DynamoDB WebSocketConnections | WS 接続 ID | U7 Infra |
| S3 training-lock-audio | Polly 音声ファイル | U7 Infra |
| Bedrock Agent | テキスト生成 | U7 Infra |
| API Gateway REST | HTTP エンドポイント | U7 Infra |
| API Gateway WebSocket | WS エンドポイント | U7 Infra |

## 共通コード（各 Lambda にコピー）

| ファイル | 内容 | コピー先 |
|---------|------|---------|
| `dynamodb_client.py` | DynamoDB CRUD ヘルパー | U2, U3, U4, U5, U6 |
| `bedrock_client.py` | Bedrock Agent 呼び出しヘルパー | U2, U3, U4, U5 |
