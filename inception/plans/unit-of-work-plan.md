# ユニット・オブ・ワーク プラン — お相撲さんと一緒

## 実行チェックリスト（全完了）
- [x] Step A: ユニット境界定義
- [x] Step B: 依存関係整理（unit-of-work-dependency.md）
- [x] Step C: ストーリーマッピング（unit-of-work-story-map.md）
- [x] Step D: 実装優先順位の決定
- [x] Step E: モノレポ構成の確定

## コンポーネント技術スタック
| コンポーネント | 技術 |
|--------------|------|
| Web UI | Next.js（React + SSR） |
| Lambda Orchestrator / Encouragement / Analysis / Dep.Tracker / WS Notifier | Python 3.12 |
| Amazon Bedrock | Claude 3 Sonnet / Haiku |
| Amazon Transcribe | Streaming API |
| Amazon Polly | SynthesizeSpeech API |
| AWS CDK | TypeScript |

## ユニット分解方針の回答
- 分割粒度: 機能別 Lambda ごとに 1 ユニット
- リポジトリ構成: モノレポ（frontend / backend / infra ディレクトリ）
- Lambda コード共有: 各 Lambda に共通コードをコピー
- CDK スタック: 1 スタックにすべてのリソースをまとめる
