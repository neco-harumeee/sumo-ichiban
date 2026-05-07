# コンポーネント依存関係 — お相撲さんと一緒

## 依存関係マトリクス

| 呼び出し元 \ 呼び出し先 | Web UI | API GW REST | API GW WS | Orchestrator | Analysis | Encouragement | Dep.Tracker | WS Notifier | Bedrock | Transcribe | Polly | S3 | DynamoDB |
|----------------------|--------|-------------|-----------|--------------|----------|---------------|-------------|-------------|---------|------------|-------|----|----------|
| **Web UI** | — | ✅ | ✅ | — | — | — | — | — | — | — | — | — | — |
| **API GW REST** | — | — | — | ✅ | — | — | — | — | — | — | — | — | — |
| **API GW WS** | — | — | — | — | — | — | — | ✅ | — | — | — | — | ✅ |
| **Orchestrator** | — | — | — | — | ✅ | ✅ | ✅ | ✅ | — | — | — | — | ✅ |
| **Analysis** | — | — | — | ✅ | — | — | — | — | — | ✅ | — | — | — |
| **Encouragement** | — | — | — | — | — | — | — | ✅ | ✅ | — | ✅ | ✅ | — |
| **Dep.Tracker** | — | — | — | — | — | — | — | — | — | — | — | — | ✅ |
| **WS Notifier** | — | — | ✅ | — | — | — | — | — | — | — | — | — | ✅ |

## 通信パターン

| パターン | 使用箇所 | 説明 |
|---------|---------|------|
| **同期 REST** | Web UI → API GW → Orchestrator | セッション開始/終了/取得 |
| **WebSocket** | Web UI ↔ API GW WS ↔ WS Notifier | リアルタイム励ましメッセージ配信 |
| **同期 Lambda 呼び出し** | Orchestrator → Analysis / Encouragement / Dep.Tracker | 各機能 Lambda を同期で呼び出し |
| **Transcribe Streaming** | Analysis Lambda → Amazon Transcribe | リアルタイム音声認識ストリーム |
| **Bedrock 呼び出し** | Encouragement Lambda → Bedrock | InvokeModel API（Claude 3 系） |
| **Polly 呼び出し** | Encouragement Lambda → Polly | SynthesizeSpeech API |
| **DynamoDB 読み書き** | Orchestrator / Dep.Tracker / WS Notifier | セッション状態・依存度スコア・接続 ID の永続化 |

## 結合度評価

| コンポーネント間 | 結合度 | 理由 |
|----------------|--------|------|
| Orchestrator ↔ 機能 Lambda | 中結合 | 同期呼び出し、インターフェースは JSON |
| Analysis ↔ Transcribe | 密結合 | ストリーミング API に依存 |
| Encouragement ↔ Bedrock | 疎結合 | API 呼び出し、プロンプトで制御 |
| Encouragement ↔ Polly | 疎結合 | API 呼び出し |
| 各 Lambda ↔ DynamoDB | 中結合 | テーブルスキーマに依存 |
| WS Notifier ↔ API GW WS | 密結合 | WebSocket 接続 ID 管理が必要 |
