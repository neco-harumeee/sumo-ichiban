# コンポーネントメソッド定義 — お相撲さんと一緒

> **注意**: 詳細なビジネスロジックは CONSTRUCTION フェーズの Functional Design で定義します。
> ここではメソッドシグネチャと高レベルの目的を定義します。

## C-04: Lambda — Orchestrator

### `start_session(event: dict) -> dict`
- **目的**: 同席セッションを開始し、Analysis Lambda を起動
- **入力**: `{ user_id, scene, started_at }`
- **出力**: `{ session_id, status, websocket_url }`

### `end_session(event: dict) -> dict`
- **目的**: セッションを終了し、依存度スコアを更新
- **入力**: `{ session_id }`
- **出力**: `{ session_id, duration_sec, encouragement_count, new_score }`

### `get_session(event: dict) -> dict`
- **目的**: セッションの現在状態を取得
- **入力**: `{ session_id }`
- **出力**: `{ session_id, status, scene, encouragement_count, elapsed_sec }`

### `handle_tension_event(event: dict) -> dict`
- **目的**: Analysis Lambda からの緊張検知イベントを受け取り、励まし生成をトリガー
- **入力**: `{ session_id, trigger_type, context }`
- **出力**: `{ triggered: bool }`

## C-05: Lambda — Encouragement

### `generate_encouragement(context: dict) -> dict`
- **目的**: 場面・検知内容を Bedrock に渡して励ましテキストを生成
- **入力**: `{ session_id, scene, trigger_type, detected_word }`
- **出力**: `{ message_text, emoji, bedrock_request_id }`

### `synthesize_audio(text: str, session_id: str) -> dict`
- **目的**: Polly で励ましテキストを MP3 に変換し S3 へ保存
- **入力**: `{ text, session_id }`
- **出力**: `{ s3_key, presigned_url, duration_seconds }`

### `deliver_encouragement(context: dict) -> dict`
- **目的**: 励ましメッセージと音声 URL を WS Notifier 経由でフロントエンドに配信
- **入力**: `{ session_id, message_text, audio_url, emoji }`
- **出力**: `{ delivered: bool, delivered_at }`

## C-06: Lambda — Analysis

### `start_transcription(session_id: str, scene: str) -> dict`
- **目的**: Transcribe ストリーミングセッションを開始
- **出力**: `{ transcription_job_id, status }`

### `detect_tension_words(transcript: str, session_id: str) -> dict`
- **目的**: 文字起こし結果から緊張ワードを検知
- **入力**: `{ transcript, session_id, scene }`
- **出力**: `{ detected: bool, trigger_type, detected_word }`

### `detect_silence(audio_chunk: bytes, session_id: str) -> dict`
- **目的**: 音声チャンクから沈黙区間を検知
- **出力**: `{ silence_detected: bool, silence_sec }`

### `analyze_speech_rate(transcript: str, duration_sec: float) -> dict`
- **目的**: 発話速度（単語/秒）を計算して早口・詰まりを検知
- **出力**: `{ words_per_sec, is_fast: bool, is_stuttering: bool }`

### `detect_filler_words(transcript: str) -> dict`
- **目的**: フィラーワード（えーと・あの・その）の連続を検知
- **出力**: `{ detected: bool, count, words: [] }`

## C-07: Lambda — Dependency Tracker

### `calculate_score(session_data: dict) -> int`
- **目的**: セッションデータから依存度スコアの増分を計算
- **入力**: `{ duration_sec, encouragement_count, scene }`
- **出力**: `score_delta: int`

### `update_score(user_id: str, delta: int) -> dict`
- **目的**: DynamoDB の依存度スコアを更新しランクを判定
- **出力**: `{ new_score, rank, total_sessions }`

### `save_session_history(session_data: dict) -> None`
- **目的**: セッション履歴を DynamoDB に保存

### `get_dependency_stats(user_id: str) -> dict`
- **目的**: 依存度統計（スコア・ランク・履歴）を取得
- **出力**: `{ score, rank, total_sessions, total_duration_sec, first_session_at }`

## C-08: Lambda — WS Notifier

### `push_update(connection_id: str, event_type: str, payload: dict) -> None`
- **目的**: 特定の WebSocket 接続にイベントをプッシュ

### `broadcast_session(session_id: str, event_type: str, payload: dict) -> None`
- **目的**: セッションに紐づく全接続にイベントをブロードキャスト

### `handle_connect(connection_id: str, session_id: str) -> None`
- **目的**: WebSocket 接続時に接続 ID を DynamoDB に保存

### `handle_disconnect(connection_id: str) -> None`
- **目的**: WebSocket 切断時に接続 ID を DynamoDB から削除

## C-09: Amazon Bedrock（Claude 3 系）

### プロンプト設計

**クレーム電話同行プロンプト**
- システム: 「あなたは力士です。相撲用語を使って短く励ましてください。」
- ユーザー: `{ scene: "phone", trigger: "tension_word", word: "申し訳" }`

**会議同席プロンプト**
- システム: 「あなたは力士です。会議中の沈黙・早口に対して短く励ましてください。」
- ユーザー: `{ scene: "meeting", trigger: "silence", silence_sec: 5 }`
