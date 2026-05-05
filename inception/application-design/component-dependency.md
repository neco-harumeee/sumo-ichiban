# コンポーネント依存関係 — TRAINING LOCK

## 依存関係マトリクス

| 呼び出し元 \ 呼び出し先 | Web UI | API GW REST | API GW WS | Orchestrator | Social Shield | Motivation | Logistics | WS Notifier | Bedrock Agent | Polly | S3 | DynamoDB |
|----------------------|--------|-------------|-----------|--------------|---------------|------------|-----------|-------------|---------------|-------|----|----------|
| **Web UI** | — | ✅ | ✅ | — | — | — | — | — | — | — | — | — |
| **API GW REST** | — | — | — | ✅ | — | — | — | — | — | — | — | — |
| **API GW WS** | — | — | — | — | — | — | — | ✅ | — | — | — | ✅ |
| **Orchestrator** | — | — | — | — | ✅ | ✅ | ✅ | ✅ | — | — | — | ✅ |
| **Social Shield** | — | — | — | — | — | — | — | ✅ | ✅ | — | — | ✅ |
| **Motivation** | — | — | — | — | — | — | — | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Logistics** | — | — | — | — | — | — | — | ✅ | ✅ | — | — | ✅ |
| **WS Notifier** | — | — | ✅ | — | — | — | — | — | — | — | — | ✅ |

---

## データフロー図

```
+------------------+
|    Web UI        |
|  (Next.js)       |
+------------------+
    |          |
    | REST      | WebSocket
    v          v
+----------+ +----------+
| API GW   | | API GW   |
| REST     | | WebSocket|
+----------+ +----------+
    |              |
    v              v
+------------------+    +------------------+
| Orchestrator     |    | WS Notifier      |
| Lambda           |    | Lambda           |
+------------------+    +------------------+
    |    |    |              ^   ^   ^
    |    |    |              |   |   |
    v    v    v              |   |   |
+------+ +----------+ +----------+
|Social| |Motivation| |Logistics |
|Shield| |Rebuttal  | |Lambda    |
|Lambda| |Lambda    | +----------+
+------+ +----------+     |
    |         |    |      |
    v         v    v      v
+------------------+  +-------+
| Bedrock Agent    |  | PA-API|
| (Claude 3)       |  | Mock  |
+------------------+  +-------+
                  |
          +-------+-------+
          |               |
          v               v
       +-------+       +-------+
       | Polly |       |  SES  |
       +-------+       | Mock  |
           |           +-------+
           v
        +-----+
        |  S3 |
        +-----+

         全 Lambda
            |
            v
       +----------+
       | DynamoDB |
       +----------+
```

---

## 通信パターン

| パターン | 使用箇所 | 説明 |
|---------|---------|------|
| **同期 REST** | Web UI → API GW → Orchestrator | セッション開始/終了/取得 |
| **WebSocket** | Web UI ↔ API GW WS ↔ WS Notifier | リアルタイム状態更新 |
| **非同期 Lambda 呼び出し** | Orchestrator → 各機能 Lambda | 機能 Lambda を非同期で起動（InvocationType=Event） |
| **EventBridge Scheduler** | → Motivation Lambda | 終了予定時刻 + X 分後にサボり検知をトリガー |
| **Bedrock Agent 呼び出し** | 各機能 Lambda → Bedrock | InvokeAgent API |
| **DynamoDB 読み書き** | 全 Lambda | セッション状態・設定・履歴の永続化 |

---

## 結合度評価

| コンポーネント間 | 結合度 | 理由 |
|----------------|--------|------|
| Orchestrator ↔ 機能 Lambda | 疎結合 | 非同期呼び出し、インターフェースは JSON |
| 機能 Lambda ↔ Bedrock Agent | 疎結合 | API 呼び出し、プロンプトで制御 |
| 機能 Lambda ↔ DynamoDB | 中結合 | テーブルスキーマに依存 |
| WS Notifier ↔ API GW WS | 密結合 | WebSocket 接続 ID 管理が必要 |
