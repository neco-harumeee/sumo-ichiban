# サービス層設計 — TRAINING LOCK

## サービス概要

TRAINING LOCK のバックエンドは **Orchestrator Lambda** を中心に、
機能別 Lambda が疎結合で連携するサービス構成です。

---

## S-01: TrainingSessionService（セッション管理サービス）

**担当コンポーネント**: C-04 Lambda Orchestrator + C-12 DynamoDB

**責務**:
- トレーニングセッションのライフサイクル管理（開始 / 進行中 / 終了）
- セッション状態の永続化と取得
- 各機能サービスの起動タイミング制御

**オーケストレーションフロー（セッション開始時）**:
```
Web UI
  → POST /session/start
  → Orchestrator Lambda
      → DynamoDB: セッション作成
      → Social Shield Lambda: 非同期起動
      → Logistics Lambda: 非同期起動
      → EventBridge: モチベ反論スケジュール登録
      → WebSocket Notifier: "session_started" プッシュ
  → レスポンス: { session_id, websocket_url }
```

---

## S-02: SocialShieldService（社会的シールドサービス）

**担当コンポーネント**: C-05 Lambda Social Shield + C-09 Bedrock Agent

**責務**:
- メール受信イベントの処理（モック）
- Bedrock Agent への文脈渡しと返信文生成
- 家族連絡のスケジュール実行
- 送信結果の DynamoDB 記録と WebSocket 通知

**オーケストレーションフロー（メール返信）**:
```
EventBridge / Orchestrator
  → Social Shield Lambda
      → Bedrock Agent: compose_email_reply(subject, body, tone)
      → SES（モック）: 返信送信
      → DynamoDB: actions_taken に追記
      → WebSocket Notifier: "shield_activated" プッシュ
```

---

## S-03: MotivationRebuttalService（モチベ反論サービス）

**担当コンポーネント**: C-06 Lambda Motivation Rebuttal + C-09 Bedrock Agent + C-10 Polly + C-11 S3

**責務**:
- サボり検知（終了予定時刻超過の判定）
- Bedrock Agent による反論テキスト生成
- Polly による音声合成と S3 保存
- メール/LINE への音声添付送信（モック）

**オーケストレーションフロー**:
```
EventBridge Scheduler（終了予定時刻 + X 分後）
  → Motivation Rebuttal Lambda
      → DynamoDB: セッション状態確認
      → detect_slacking(): True の場合のみ続行
      → Bedrock Agent: generate_rebuttal_text(pressure_level)
      → Polly: synthesize_speech(text) → MP3
      → S3: MP3 保存 + 署名付き URL 発行
      → SES/LINE（モック）: 音声添付メール送信
      → DynamoDB: actions_taken に追記
      → WebSocket Notifier: "rebuttal_sent" プッシュ
```

---

## S-04: LogisticsService（物流連携サービス）

**担当コンポーネント**: C-07 Lambda Logistics + C-09 Bedrock Agent

**責務**:
- Bedrock Agent によるギア発注必要性の判断
- PA-API（モック）での商品検索・発注
- 発注通知の送信と履歴記録

**オーケストレーションフロー**:
```
Orchestrator Lambda（セッション開始時）
  → Logistics Lambda
      → Bedrock Agent: decide_gear_order(session_history)
      → should_order == True の場合:
          → PA-API（モック）: 商品検索 → 発注
          → DynamoDB: OrderHistory に記録
          → SES（モック）: "ぽちっとしておきました" 通知
          → WebSocket Notifier: "order_placed" プッシュ
```

---

## S-05: RealtimeNotificationService（リアルタイム通知サービス）

**担当コンポーネント**: C-08 Lambda WebSocket Notifier + C-03 API Gateway WebSocket + C-12 DynamoDB

**責務**:
- WebSocket 接続の管理（接続 ID の DynamoDB 保存）
- 各サービスからのイベントを Web UI へリアルタイムプッシュ
- 接続切断時のクリーンアップ

**WebSocket イベント一覧**:
| イベント | 送信元 | ペイロード |
|---------|--------|-----------|
| `session_started` | Orchestrator | `{ session_id, end_time }` |
| `shield_activated` | Social Shield | `{ action_type, recipient, sent_at }` |
| `rebuttal_sent` | Motivation | `{ rebuttal_text, audio_url, sent_at }` |
| `order_placed` | Logistics | `{ item, price, order_id }` |
| `session_ended` | Orchestrator | `{ summary, actions_count }` |
