# サービス層設計 — お相撲さんと一緒

お相撲さんと一緒のバックエンドは **Orchestrator Lambda** を中心に、機能別 Lambda が疎結合で連携するサービス構成です。

## S-01: SessionService（セッション管理サービス）
**担当**: C-04 Lambda Orchestrator + C-13 DynamoDB
- セッションのライフサイクル管理（開始 / 進行中 / 終了）
- 各機能サービスの起動タイミング制御
- WebSocket 接続 ID の管理

## S-02: AnalysisService（緊張検知サービス）
**担当**: C-06 Lambda Analysis + C-10 Amazon Transcribe
- Transcribe によるリアルタイム音声認識
- 緊張ワード検知（「申し訳」「すみません」「困ります」等）
- 沈黙検知（3秒以上の無音区間）
- 発話速度解析（単語/秒の計算による早口・詰まり検知）
- フィラーワード検知（「えーと」「あの」「その」の連続）

## S-03: EncouragementService（励まし生成サービス）
**担当**: C-05 Lambda Encouragement + C-09 Amazon Bedrock + C-11 Amazon Polly + C-12 Amazon S3
- Bedrock（Claude 3 系）による励ましテキスト生成
- 相撲用語・力士らしい言い回しの使用
- 場面（クレーム電話 / 会議 / 相談）に応じたトーン調整
- Polly による音声ファイル（MP3）生成
- S3 への音声ファイル一時保存

## S-04: DependencyTrackerService（依存度管理サービス）
**担当**: C-07 Lambda Dependency Tracker + C-13 DynamoDB
- セッション終了時の依存度スコア計算
- 利用回数・利用時間・緊張検知回数からスコアを算出
- 依存度ランク更新（序ノ口 → 序二段 → 三段目 → 幕下 → 十両 → 幕内 → 横綱）
- 利用履歴の永続化

## S-05: RealtimeNotificationService（リアルタイム通知サービス）
**担当**: C-08 Lambda WS Notifier + C-03 API Gateway WebSocket + C-13 DynamoDB

WebSocket イベント一覧:
| イベント | 送信元 | ペイロード |
|---------|--------|-----------|
| `session_started` | Orchestrator | `{ session_id, scene, started_at }` |
| `tension_detected` | Analysis | `{ trigger_type, detected_word, silence_sec }` |
| `encouragement_ready` | Encouragement | `{ message_text, audio_url, emoji }` |
| `dependency_updated` | Dependency Tracker | `{ score, rank, total_sessions }` |
| `session_ended` | Orchestrator | `{ duration_sec, encouragement_count, new_score }` |
