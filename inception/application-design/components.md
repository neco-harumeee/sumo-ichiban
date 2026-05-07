# コンポーネント定義 — お相撲さんと一緒

## コンポーネント一覧

| ID | コンポーネント | 技術 | 役割 |
|----|--------------|------|------|
| C-01 | Web UI | Next.js (React + SSR) | 場面選択・お相撲さん表示・依存度ダッシュボード |
| C-02 | API Gateway REST | Amazon API Gateway | Web UI からの REST リクエスト受付 |
| C-03 | API Gateway WebSocket | Amazon API Gateway | リアルタイム励ましメッセージ配信 |
| C-04 | Lambda Orchestrator | Python 3.12 | 各 Lambda の制御・フロー管理 |
| C-05 | Lambda Encouragement | Python 3.12 | Bedrock による励ましテキスト生成・Polly 音声生成 |
| C-06 | Lambda Analysis | Python 3.12 | Transcribe 音声解析（緊張ワード検知・沈黙検知・発話速度解析） |
| C-07 | Lambda Dependency Tracker | Python 3.12 | 依存度スコア計算・DynamoDB 更新 |
| C-08 | Lambda WS Notifier | Python 3.12 | WebSocket 経由でリアルタイム通知をプッシュ |
| C-09 | Amazon Bedrock | Claude 3 Sonnet/Haiku | 励ましテキスト生成（相撲用語・状況対応） |
| C-10 | Amazon Transcribe | — | リアルタイム音声認識・緊張ワード検知・沈黙検知・発話速度解析 |
| C-11 | Amazon Polly | — | お相撲さん音声合成（MP3 出力） |
| C-12 | Amazon S3 | — | 生成した音声ファイルの一時保存 |
| C-13 | Amazon DynamoDB | — | セッション状態・依存度スコア・利用履歴管理 |
| C-14 | AWS CDK | TypeScript | インフラ全体の IaC 管理 |

---

## コンポーネント詳細

### C-01: Web UI（Next.js）
- **役割**: ユーザーインターフェース全体
- **主要画面**:
  - ホーム（場面選択）
  - 同席モード（お相撲さん表示・励ましメッセージ）
  - 依存度ダッシュボード
  - 利用履歴
- **通信**: REST（API Gateway）+ WebSocket（リアルタイム更新）

### C-04: Lambda Orchestrator
- **役割**: セッション管理・各 Lambda の呼び出し制御
- **主要処理**:
  - セッション開始/終了
  - 分析結果の受信と励まし生成のトリガー
  - 依存度スコアの更新指示

### C-05: Lambda Encouragement
- **役割**: 励ましコンテンツの生成
- **主要処理**:
  - Bedrock に状況（場面・検知内容）を渡して励ましテキスト生成
  - Polly で音声ファイル生成
  - S3 に音声ファイル保存
  - WS Notifier 経由でフロントエンドに配信

### C-06: Lambda Analysis
- **役割**: ユーザーの緊張状態を検知
- **主要処理**:
  - Transcribe でリアルタイム音声認識・緊張ワード検知・沈黙検知
  - 発話速度（単語/秒）を計算して早口・詰まりを検知
  - 検知結果を Orchestrator に通知

### C-07: Lambda Dependency Tracker
- **役割**: 依存度の計算と記録
- **主要処理**:
  - セッション終了時に依存度スコアを計算
  - DynamoDB に利用履歴・スコアを保存
  - 依存度ランク（序ノ口〜横綱）を更新
