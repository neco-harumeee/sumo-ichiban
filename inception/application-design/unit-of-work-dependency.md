# ユニット依存関係 — お相撲さんと一緒

## 依存関係マトリクス

| 依存元 \ 依存先 | U1 Frontend | U2 Orchestrator | U3 Encouragement | U4 Analysis | U5 Dep.Tracker | U6 Infra |
|---------------|-------------|-----------------|------------------|-------------|----------------|----------|
| **U1 Frontend** | — | ✅ REST | — | — | — | ✅ デプロイ先 |
| **U2 Orchestrator** | — | — | ✅ 同期呼び出し | ✅ 同期呼び出し | ✅ 同期呼び出し | ✅ リソース定義 |
| **U3 Encouragement** | — | — | — | — | — | ✅ リソース定義 |
| **U4 Analysis** | — | ✅ 通知 | — | — | — | ✅ リソース定義 |
| **U5 Dep.Tracker** | — | — | — | — | — | ✅ リソース定義 |
| **U6 Infra** | — | — | — | — | — | — |

## 開発順序（推奨）
- Phase 1（並行可能）: U6 Infra + U3 Encouragement（コアバリュー）
- Phase 2（U6 完了後）: U2 Orchestrator + U4 Analysis
- Phase 3（U2 完了後）: U1 Frontend + U5 Dep.Tracker
- Phase 4: 統合テスト・デモ動画撮影

## 共有 AWS リソース（全ユニットが依存）
DynamoDB（sessions / dependency_scores / ws_connections）、S3（sumo-audio）、API Gateway REST/WebSocket（すべて U6 Infra 管理）

## 共通コード（各 Lambda にコピー）
- `dynamodb_client.py` → U2, U4, U5
- `bedrock_client.py` → U3
- `ws_notifier_client.py` → U2, U3, U4
