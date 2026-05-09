# アプリケーション設計 — お相撲さんと一緒
## 設計サマリー

| 項目 | 決定内容 |
|------|---------|
| **フロントエンド** | Next.js（React + SSR） |
| **バックエンド** | 機能別 Lambda（Orchestrator + Encouragement + Analysis + Dependency Tracker + WS Notifier） |
| **AI エンジン** | Amazon Bedrock（Claude 3 Sonnet / Haiku） |
| **音声認識・状況検知** | Amazon Transcribe（リアルタイム） |
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
|  - 場面選択画面（仕事 / 日常生活）                 |
|  - お相撲さん同席モード（助言メッセージ表示）        |
|  - 依存度ダッシュボード（番付表示）                 |
|  - 利用履歴（仕事/日常シーン別）                   |
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

### 仕事サポートフロー（押し出し・寄り切り・突き押し）
1. ユーザーが仕事シーン（クレーム電話等）を選択 → Orchestrator がセッション開始
2. マイク音声 → Analysis Lambda → Transcribe でリアルタイム認識
3. 緊張ワード / 沈黙 / 早口検知 → Orchestrator に通知
4. Orchestrator → Encouragement Lambda → Bedrock で助言テキスト生成（相撲技を使った実用的アドバイス）
5. Encouragement Lambda → Polly で音声生成 → S3 保存
6. WS Notifier → WebSocket → フロントエンドに助言メッセージ + 音声 URL 配信
7. セッション終了 → Dependency Tracker → DynamoDB に記録・番付更新

### 日常生活サポートフロー（引き落とし・叩き込み・割り出し）
1. ユーザーが日常シーン（献立・買い物・体調等）を選択 → Orchestrator がセッション開始
2. テキスト入力または音声入力 → Analysis Lambda → 迷い/疲労ワード検知
3. 検知内容 → Orchestrator に通知
4. Orchestrator → Encouragement Lambda → Bedrock で実用的な提案テキスト生成
5. 以降は仕事サポートフローと同様

---

## DynamoDB テーブル設計

### sessions テーブル
| 属性 | 型 | 説明 |
|------|-----|------|
| session_id (PK) | String | セッション ID |
| user_id | String | ユーザー ID（デモ用固定値） |
| scene | String | 場面（phone/meeting/meal/shopping/health/schedule/relationship 等） |
| scene_category | String | シーンカテゴリ（work / daily） |
| sumo_technique | String | 対応する相撲技（oshi_dashi/yori_kiri/hikiotoshi 等） |
| started_at | String | 開始時刻（ISO 8601） |
| ended_at | String | 終了時刻 |
| duration_sec | Number | セッション時間（秒） |
| support_count | Number | サポート回数 |
| ws_connection_id | String | WebSocket 接続 ID |

### dependency_scores テーブル
| 属性 | 型 | 説明 |
|------|-----|------|
| user_id (PK) | String | ユーザー ID |
| score | Number | 依存度スコア（0〜100） |
| rank | String | 番付（序ノ口〜横綱） |
| total_sessions | Number | 累計セッション数 |
| work_sessions | Number | 仕事シーン累計 |
| daily_sessions | Number | 日常シーン累計 |
| total_duration_sec | Number | 累計利用時間 |
| first_session_at | String | 初回利用日時（初めて四つに組んだ日） |
| last_session_at | String | 最終利用日時 |

### ws_connections テーブル
| 属性 | 型 | 説明 |
|------|-----|------|
| connection_id (PK) | String | WebSocket 接続 ID |
| user_id | String | ユーザー ID |
| connected_at | String | 接続時刻 |

---

## 相撲技とシーンのマッピング

| 相撲技 | 技の意味 | 対応シーン | 助言スタイル |
|--------|---------|-----------|------------|
| 押し出し（oshi_dashi） | 相手を押して土俵の外へ | クレーム電話 | 前に進む・圧力を受け流す |
| 寄り切り（yori_kiri） | じわじわ寄って勝つ | 1on1での交渉 | 相手が押されていることに気づかない技 |
| 突き押し（tsuki_oshi） | 素早い突きで攻める | Zoom 会議 | タイミングを掴む・発言する |
| 掬い投げ（sukui_nage） | 相手の懐に入る | 苦手な相談 | 相手の立場を理解する |
| 上手投げ（uwate_nage） | 大きく状況を動かす | 気まずい報告 | 一気に状況を変える |
| 引き落とし（hiki_otoshi） | 迷いを引いて断ち切る | 献立決め | 迷いを断ち切る・即決する |
| 叩き込み（hataki_komi） | 確実に押さえる | 買い物リスト | 漏れなく確実に管理する |
| 土俵際の踏ん張り | 崩れそうで踏ん張る | 体調管理 | 体調を立て直す・無理しない |
| 割り出し（waridashi） | 込み入った状況を整理 | スケジュール調整 | 優先順位をつける・整理する |
| 抱え込み（kakaekomi） | 相手ごと受け止める | 人間関係の悩み | 共感・受け止める・一緒に考える |

---

## 主要設計決定と根拠

| 決定 | 根拠 |
|------|------|
| 仕事・日常の両シーン対応 | 依存度を高めるには生活全般をカバーする必要がある。仕事だけでは離脱率が高い |
| 相撲技とシーンのマッピング | 技の特性がシーンの本質を表現しており、ユーザーが直感的に理解できる |
| 機能別 Lambda 分割 | 各機能が独立してスケール・デプロイ可能。ハッカソンでの部分的なデモも容易 |
| Bedrock 直接呼び出し（Agents なし） | 助言テキスト生成はシンプルなプロンプトで十分。Agents は不要 |
| REST + WebSocket 併用 | 操作は REST でシンプルに、助言メッセージは WebSocket でリアルタイム表示 |
| DynamoDB 永続化 | デモ中にページリロードしても番付（依存度スコア）が保持される |
| AWS CDK | インフラをコードで管理し、ハッカソン後の再現・共有が容易 |
| scene_category 属性追加 | 仕事/日常シーンの利用比率を分析し、依存パターンを可視化するため |
