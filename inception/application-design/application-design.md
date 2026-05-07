# アプリケーション設計 — お相撲さんと一緒

## 設計サマリー

| 項目 | 決定内容 |
|------|---------|
| **フロントエンド** | Next.js（React + SSR） |
| **バックエンド** | 機能別 Lambda（Orchestrator + Encouragement + Analysis + Dependency Tracker + WS Notifier） |
| **AI エンジン** | Amazon Bedrock（Claude 3 Sonnet / Haiku） |
| **音声認識・緊張検知** | Amazon Transcribe（リアルタイム） |
| **音声生成** | Amazon Polly → S3 |
| **状態管理** | Amazon DynamoDB（3 テーブル） |
| **通信** | REST（API Gateway）+ WebSocket（API Gateway WebSocket） |
| **インフラ管理** | AWS CDK（TypeScript） |

---

## アーキテクチャ概要

```
+--------------------------------------------------+
|  FRONTEND                                        |
|  Next.js (Vercel / S3+CloudFront)                |
|  - 場面選択画面                                   |
|  - お相撲さん同席モード（励ましメッセージ表示）      |
|  - 依存度ダッシュボード                            |
|  - 利用履歴                                      |
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
    |      |      |         ^    ^
    |      |      |         |    |
    v      v      v         |    |
+----------+ +----------+ +-----+
|Encourage-| |Analysis  | |Dep. |
|ment      | |Lambda    | |Track|
|Lambda    | |(Transcr.)| |er   |
+----------+ +----------+ +-----+
    |              |         |
    v              v         v
+-------+    +-------+  +----------+
|Bedrock|    |Transcr|  | DynamoDB |
|Claude |    |ibe    |  | Sessions |
+-------+    +-------+  | Scores   |
    |                   | History  |
    v                   +----------+
+-------+
| Polly |
+-------+
    |
    v
  +----+
  | S3 |
  | MP3|
  +----+

  インフラ全体
      |
      v
+----------+
| AWS CDK  |
|(TypeScript)|
+----------+
```

---

## データフロー

### クレーム電話同行フロー
1. ユーザーが「クレーム電話」を選択 → Orchestrator がセッション開始
2. マイク音声 → Analysis Lambda → Transcribe でリアルタイム認識
3. 緊張ワード / 沈黙検知 → Orchestrator に通知
4. Orchestrator → Encouragement Lambda → Bedrock で励ましテキスト生成
5. Encouragement Lambda → Polly で音声生成 → S3 保存
6. WS Notifier → WebSocket → フロントエンドに励ましメッセージ + 音声 URL 配信
7. セッション終了 → Dependency Tracker → DynamoDB に記録

### 会議同席フロー
1. ユーザーが「Zoom会議」を選択 → Orchestrator がセッション開始
2. マイク音声 → Analysis Lambda → Transcribe でリアルタイム認識
3. 緊張ワード頻出 / 沈黙 / 発話速度上昇 / 長時間ミュート検知 → Orchestrator に通知
4. 以降はクレーム電話フローと同様

---

## DynamoDB テーブル設計

### sessions テーブル
| 属性 | 型 | 説明 |
|------|-----|------|
| session_id (PK) | String | セッション ID |
| user_id | String | ユーザー ID（デモ用固定値） |
| scene | String | 場面（phone/meeting/consultation 等） |
| started_at | String | 開始時刻（ISO 8601） |
| ended_at | String | 終了時刻 |
| duration_sec | Number | セッション時間（秒） |
| encouragement_count | Number | 励まし回数 |
| ws_connection_id | String | WebSocket 接続 ID |

### dependency_scores テーブル
| 属性 | 型 | 説明 |
|------|-----|------|
| user_id (PK) | String | ユーザー ID |
| score | Number | 依存度スコア（0〜100） |
| rank | String | 依存度ランク（序ノ口〜横綱） |
| total_sessions | Number | 累計セッション数 |
| total_duration_sec | Number | 累計利用時間 |
| first_session_at | String | 初回利用日時 |
| last_session_at | String | 最終利用日時 |

### ws_connections テーブル
| 属性 | 型 | 説明 |
|------|-----|------|
| connection_id (PK) | String | WebSocket 接続 ID |
| user_id | String | ユーザー ID |
| connected_at | String | 接続時刻 |

---

## 主要設計決定と根拠

| 決定 | 根拠 |
|------|------|
| 機能別 Lambda 分割 | 各機能が独立してスケール・デプロイ可能。ハッカソンでの部分的なデモも容易 |
| Bedrock 直接呼び出し（Agents なし） | 励ましテキスト生成はシンプルなプロンプトで十分。Agents は不要 |
| REST + WebSocket 併用 | 操作は REST でシンプルに、励ましメッセージは WebSocket でリアルタイム表示 |
| DynamoDB 永続化 | デモ中にページリロードしても依存度スコアが保持される |
| AWS CDK | インフラをコードで管理し、ハッカソン後の再現・共有が容易 |
| Rekognition モック削除 | リアルタイム表情分析はブラウザ→AWS間のレイテンシが高く実用的でない。Transcribe の音声解析で十分な緊張検知が可能 |
