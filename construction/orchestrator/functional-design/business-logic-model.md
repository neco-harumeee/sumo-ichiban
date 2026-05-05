# ビジネスロジックモデル — Unit 2: Orchestrator

## フロー 1: セッション開始（start_session）

```
入力: { user_id, end_time_planned, feature_flags }

1. 既存アクティブセッションの確認
   └─ DynamoDB: user_id で ACTIVE セッションを検索
       ├─ 存在する場合:
       │   ├─ 既存セッションを CANCELLED に更新
       │   └─ WebSocket 通知: "session_cancelled"
       └─ 存在しない場合: 続行

2. 新規セッション作成
   ├─ session_id = UUID 生成
   ├─ status = ACTIVE
   ├─ start_time = 現在時刻
   ├─ end_time_planned = 入力値
   ├─ enabled_features = 入力の feature_flags
   └─ DynamoDB: Sessions テーブルに保存

3. 有効な機能 Lambda を条件付き起動（非同期）
   ├─ social_shield_enabled == true → Social Shield Lambda 起動
   └─ logistics_enabled == true    → Logistics Lambda 起動

4. EventBridge Scheduler 登録
   └─ motivation_rebuttal_enabled == true の場合のみ
       └─ スケジュール: end_time_planned に Motivation Lambda を起動

5. WebSocket 通知
   └─ "session_started" イベントをブロードキャスト

6. レスポンス返却
   └─ { session_id, status: "ACTIVE", end_time_planned }
```

## フロー 2: セッション終了（end_session）

```
入力: { session_id }

1. セッション取得・検証
   └─ DynamoDB: session_id でセッションを取得
       └─ ACTIVE でない場合: エラー返却

2. セッション状態更新
   ├─ status = COMPLETED
   ├─ end_time_actual = 現在時刻
   └─ DynamoDB: 更新保存

3. セッションサマリー生成
   ├─ duration_minutes = (end_time_actual - start_time) / 60
   ├─ actions_count = actions_taken.length
   └─ SessionSummary オブジェクト生成

4. WebSocket 通知
   └─ "session_ended" イベント + サマリーをブロードキャスト

5. レスポンス返却
   └─ { session_id, summary: SessionSummary }
```

## フロー 3: セッション状態取得（get_session）

```
入力: { session_id }

1. DynamoDB: session_id でセッションを取得
2. 存在しない場合: 404 エラー返却
3. レスポンス返却
   └─ { session_id, status, actions_taken, elapsed_minutes }
```

## フロー 4: アクション記録（record_action）
※ 各機能 Lambda から呼び出される内部フロー

```
入力: { session_id, action_type, result, mock }

1. DynamoDB: session_id のセッションを取得
2. actions_taken に ActionRecord を追加
3. updated_at を更新
4. DynamoDB: 保存
```

## 状態遷移図

```
[新規]
  |
  | start_session()
  v
[ACTIVE] ──────────────────────────────→ [CANCELLED]
  |                                        (重複起動時の既存セッション)
  | end_session()
  v
[COMPLETED]
```
