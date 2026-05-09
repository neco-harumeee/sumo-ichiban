# サービス層設計 — 相撲一番（SUMO ICHIBAN）

相撲一番のバックエンドは **Orchestrator Lambda** を中心に、機能別 Lambda が疎結合で連携するサービス構成です。  
各サービスは相撲の「技」に対応し、場面に応じた最適な技を繰り出します。

---

## S-01: SessionService（仕切りサービス）
**担当**: C-04 Lambda Orchestrator + C-13 DynamoDB  
**相撲技**: 仕切り（試合開始の準備）

- セッションのライフサイクル管理（開始 / 進行中 / 終了）
- 仕事シーン・日常シーンの判別と対応する相撲技の選択
- 各機能サービスの起動タイミング制御
- WebSocket 接続 ID の管理

**対応シーン**:
| シーン | 相撲技 | 起動する技 |
|--------|--------|-----------|
| クレーム電話 | 押し出し | AnalysisService（緊張ワード検知） |
| 1on1での交渉 | 寄り切り | AnalysisService（緊張ワード検知） |
| Zoom 会議 | 突き押し | AnalysisService（沈黙・早口検知） |
| 献立決め | 引き落とし | EncouragementService（直接提案） |
| 買い物リスト | 叩き込み | EncouragementService（リスト管理） |
| 体調管理 | 土俵際の踏ん張り | EncouragementService（健康アドバイス） |
| スケジュール調整 | 割り出し | EncouragementService（優先順位整理） |
| 人間関係の悩み | 抱え込み | EncouragementService（共感・アドバイス） |

---

## S-02: AnalysisService（状況検知サービス）
**担当**: C-06 Lambda Analysis + C-10 Amazon Transcribe  
**相撲技**: 見合い（相手の動きを読む）

- Transcribe によるリアルタイム音声認識
- **緊張ワード検知**（「申し訳」「すみません」「困ります」等）→ 押し出し技発動
- **迷いワード検知**（「どうしよう」「わからない」「決められない」等）→ 引き落とし技発動
- **疲労ワード検知**（「疲れた」「しんどい」「もう無理」等）→ 土俵際の踏ん張り技発動
- **沈黙検知**（3秒以上の無音区間）→ 助け舟を出す
- **発話速度解析**（単語/秒の計算による早口・詰まり検知）→ 突き押し技発動
- **フィラーワード検知**（「えーと」「あの」「その」の連続）→ 寄り切り技発動

---

## S-03: EncouragementService（助言・励まし生成サービス）
**担当**: C-05 Lambda Encouragement + C-09 Amazon Bedrock + C-11 Amazon Polly + C-12 Amazon S3  
**相撲技**: 決まり手（勝負を決める一手）

- Bedrock（Claude 3 系）による助言・励ましテキスト生成
- **相撲技・相撲用語を使った実用的なアドバイス**
- シーンカテゴリ（仕事 / 日常）に応じたトーン調整:
  - 仕事シーン: 具体的な対処法・フレーズ提案（「『少々お時間をいただけますか』と言いましょう」等）
  - 日常シーン: 実用的な提案・選択肢の提示（「冷蔵庫の食材でこの献立はいかがですか」等）
- Polly による音声ファイル（MP3）生成
- S3 への音声ファイル一時保存（TTL: 1時間）

**プロンプト設計方針**:
- 相撲技の名前と意味を文脈に組み込む
- 実用的な情報（具体的フレーズ・手順・選択肢）を必ず含める
- 過度に依存を促すのではなく「一緒に考える」スタンスを維持
- 医療・法律に関わる内容は専門家への相談を促す

---

## S-04: DependencyTrackerService（番付管理サービス）
**担当**: C-07 Lambda Dependency Tracker + C-13 DynamoDB  
**相撲技**: 番付昇進（実力に応じた地位の上昇）

- セッション終了時の依存度スコア計算
- 利用回数・利用時間・サポート要求回数からスコアを算出
- **番付更新**（序ノ口 → 序二段 → 三段目 → 幕下 → 十両 → 幕内 → 横綱）
- 仕事シーン・日常シーン別の利用履歴の永続化
- 番付昇進時の特別メッセージ生成

**番付スコア計算式**:
```
score += セッション数 × 2
score += 利用時間（分）× 0.5
score += サポート要求回数 × 1
score += 日常シーン利用 × 1.5  # 日常依存は加速度的に深まる
```

---

## S-05: RealtimeNotificationService（リアルタイム通知サービス）
**担当**: C-08 Lambda WS Notifier + C-03 API Gateway WebSocket + C-13 DynamoDB  
**相撲技**: 呼び出し（タイミングよく場を整える）

WebSocket イベント一覧:
| イベント | 送信元 | ペイロード |
|---------|--------|-----------|
| `session_started` | Orchestrator | `{ session_id, scene, scene_category, sumo_technique, started_at }` |
| `situation_detected` | Analysis | `{ trigger_type, detected_word, silence_sec, sumo_technique }` |
| `advice_ready` | Encouragement | `{ message_text, audio_url, emoji, sumo_technique }` |
| `dependency_updated` | Dependency Tracker | `{ score, rank, total_sessions, work_sessions, daily_sessions }` |
| `session_ended` | Orchestrator | `{ duration_sec, support_count, new_score, new_rank }` |
