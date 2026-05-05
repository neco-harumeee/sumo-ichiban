# コンポーネントメソッド定義 — TRAINING LOCK

> **注意**: 詳細なビジネスロジックは CONSTRUCTION フェーズの Functional Design で定義します。
> ここではメソッドシグネチャと高レベルの目的を定義します。

---

## C-04: Lambda — Orchestrator

### `start_session(event: dict) -> dict`
- **目的**: トレーニングセッションを開始し、各機能 Lambda を起動
- **入力**: `{ user_id, start_time, end_time, settings }`
- **出力**: `{ session_id, status, websocket_url }`

### `end_session(event: dict) -> dict`
- **目的**: セッションを終了し、実行サマリーを返す
- **入力**: `{ session_id }`
- **出力**: `{ session_id, actions_taken: [], summary }`

### `get_session(event: dict) -> dict`
- **目的**: セッションの現在状態を取得
- **入力**: `{ session_id }`
- **出力**: `{ session_id, status, actions_taken: [], elapsed_time }`

---

## C-05: Lambda — Social Shield

### `generate_email_reply(context: dict) -> dict`
- **目的**: 受信メールの文脈を Bedrock Agent に渡し、自動返信文を生成・送信（モック）
- **入力**: `{ session_id, email_subject, email_body, tone_setting }`
- **出力**: `{ reply_text, sent_at, mock: bool }`

### `send_family_notification(context: dict) -> dict`
- **目的**: 家族への定期連絡文を生成・送信（モック）
- **入力**: `{ session_id, recipient_email, current_time, session_end_time }`
- **出力**: `{ message_text, sent_at, mock: bool }`

---

## C-06: Lambda — Motivation Rebuttal

### `detect_slacking(session_id: str) -> bool`
- **目的**: 終了予定時刻を過ぎているかを判定
- **入力**: `session_id`
- **出力**: `True / False`

### `generate_rebuttal(context: dict) -> dict`
- **目的**: Bedrock Agent でサボり反論テキストを生成
- **入力**: `{ session_id, pressure_level, reason_hint }`
- **出力**: `{ rebuttal_text, pressure_level }`

### `synthesize_audio(text: str, session_id: str) -> dict`
- **目的**: Polly で反論テキストを MP3 に変換し S3 へ保存
- **入力**: `text, session_id`
- **出力**: `{ s3_key, presigned_url, duration_seconds }`

### `send_rebuttal(context: dict) -> dict`
- **目的**: 音声ファイルをメール/LINE に添付して送信（モック）
- **入力**: `{ session_id, presigned_url, rebuttal_text, recipient }`
- **出力**: `{ sent_at, channel, mock: bool }`

---

## C-07: Lambda — Logistics

### `evaluate_gear_need(session_id: str) -> dict`
- **目的**: Bedrock Agent がギア発注の必要性を判断
- **入力**: `session_id`
- **出力**: `{ should_order: bool, item_name, reason }`

### `search_product(item_name: str) -> dict`
- **目的**: PA-API（モック）で商品を検索
- **入力**: `item_name`
- **出力**: `{ asin, title, price, url }`

### `place_order(product: dict, session_id: str) -> dict`
- **目的**: PA-API（モック）で発注し、通知を送信
- **入力**: `{ asin, title, price }, session_id`
- **出力**: `{ order_id, item, price, notified_at, mock: bool }`

---

## C-08: Lambda — WebSocket Notifier

### `push_update(connection_id: str, event_type: str, payload: dict) -> None`
- **目的**: WebSocket 経由で Web UI にリアルタイム更新を送信
- **入力**: `connection_id, event_type (shield_activated|rebuttal_sent|order_placed), payload`
- **出力**: なし（fire-and-forget）

### `broadcast_session(session_id: str, event_type: str, payload: dict) -> None`
- **目的**: セッションに紐づく全接続へブロードキャスト
- **入力**: `session_id, event_type, payload`
- **出力**: なし

---

## C-09: Bedrock Agent

### Action Group: `social-shield-actions`
- `compose_email_reply(subject, body, tone)` → 返信文テキスト
- `compose_family_message(time_of_day, end_time)` → 家族連絡文テキスト

### Action Group: `motivation-actions`
- `generate_rebuttal_text(pressure_level, context)` → 反論テキスト

### Action Group: `logistics-actions`
- `decide_gear_order(session_history)` → 発注判断と商品名

---

## C-12: DynamoDB — テーブル設計（概要）

### Sessions テーブル
| PK | SK | 属性 |
|----|-----|------|
| `session_id` | — | `user_id, start_time, end_time, status, actions_taken[]` |

### UserSettings テーブル
| PK | SK | 属性 |
|----|-----|------|
| `user_id` | — | `tone_setting, pressure_level, family_email, family_schedule` |

### OrderHistory テーブル
| PK | SK | 属性 |
|----|-----|------|
| `user_id` | `order_id` | `item, price, asin, ordered_at, session_id` |

### WebSocketConnections テーブル
| PK | SK | 属性 |
|----|-----|------|
| `connection_id` | — | `session_id, connected_at` |
