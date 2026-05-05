# ドメインエンティティ — Unit 2: Orchestrator

## エンティティ一覧

### TrainingSession（トレーニングセッション）

| フィールド | 型 | 説明 |
|-----------|-----|------|
| `session_id` | string (UUID) | セッション一意識別子 |
| `user_id` | string | ユーザー識別子（デモ用固定値可） |
| `status` | SessionStatus | セッション状態 |
| `start_time` | ISO8601 datetime | 開始時刻 |
| `end_time_planned` | ISO8601 datetime | 終了予定時刻 |
| `end_time_actual` | ISO8601 datetime \| null | 実際の終了時刻 |
| `enabled_features` | FeatureFlags | 有効化された機能フラグ |
| `actions_taken` | ActionRecord[] | 実行済みアクション一覧 |
| `created_at` | ISO8601 datetime | レコード作成時刻 |
| `updated_at` | ISO8601 datetime | 最終更新時刻 |

### SessionStatus（セッション状態）

```
ACTIVE    - トレーニング中（シールド展開中）
COMPLETED - 正常終了（ユーザーが終了宣言）
CANCELLED - 強制終了（重複起動による自動終了）
```

### FeatureFlags（機能フラグ）

| フィールド | 型 | デフォルト | 説明 |
|-----------|-----|-----------|------|
| `social_shield_enabled` | bool | true | 社会的シールドを有効化 |
| `motivation_rebuttal_enabled` | bool | true | モチベ反論を有効化 |
| `logistics_enabled` | bool | true | 物流連携を有効化 |

### ActionRecord（実行済みアクション）

| フィールド | 型 | 説明 |
|-----------|-----|------|
| `action_type` | ActionType | アクション種別 |
| `executed_at` | ISO8601 datetime | 実行時刻 |
| `result` | string | 実行結果の概要 |
| `mock` | bool | モック実行かどうか |

### ActionType（アクション種別）

```
EMAIL_REPLY_SENT      - メール自動返信
FAMILY_NOTIFIED       - 家族への連絡
REBUTTAL_SENT         - モチベ反論送信
ORDER_PLACED          - 物流発注
```

### SessionSummary（セッションサマリー）

| フィールド | 型 | 説明 |
|-----------|-----|------|
| `session_id` | string | セッション ID |
| `duration_minutes` | int | トレーニング時間（分） |
| `actions_count` | int | 実行アクション総数 |
| `actions_taken` | ActionRecord[] | 実行済みアクション一覧 |

## エンティティ関係

```
TrainingSession
    |
    +-- FeatureFlags (1:1)
    |
    +-- ActionRecord[] (1:N)
    |
    +-- SessionSummary (1:1, 終了時生成)
```
