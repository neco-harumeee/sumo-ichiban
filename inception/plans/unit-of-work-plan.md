# ユニット・オブ・ワーク プラン — TRAINING LOCK

## 実行チェックリスト

- [x] Step A: ユニット境界の定義（unit-of-work.md）
- [x] Step B: ユニット依存関係の定義（unit-of-work-dependency.md）
- [x] Step C: ストーリーマッピング（unit-of-work-story-map.md）
- [x] Step D: コード構成戦略の文書化（unit-of-work.md に追記）
- [x] Step E: ユニット境界の検証

---

## 背景

Application Design で以下のコンポーネントが定義されました：

| コンポーネント | 技術 |
|--------------|------|
| Web UI | Next.js |
| Lambda Orchestrator | Python 3.12 |
| Lambda Social Shield | Python 3.12 |
| Lambda Motivation Rebuttal | Python 3.12 |
| Lambda Logistics | Python 3.12 |
| Lambda WS Notifier | Python 3.12 |
| Bedrock Agent | Claude 3 |
| AWS CDK | TypeScript |

---

## 質問 — ユニット分解方針の確認

以下の質問に回答してください。各 `[Answer]:` タグの後に選択肢の文字を記入してください。

---

### 質問 1: ユニットの分割粒度
開発ユニットをどの粒度で分割しますか？

A) 機能別 Lambda ごとに 1 ユニット（Social Shield / Motivation / Logistics / Orchestrator それぞれ独立）
B) フロントエンド / バックエンド / インフラの 3 ユニット
C) フロントエンド / コア Lambda 群 / インフラ の 3 ユニット（Lambda は 1 リポジトリにまとめる）
D) 全部 1 ユニット（モノレポ、ハッカソン向けシンプル構成）
X) その他（[Answer]: の後に記述してください）

[Answer]: A) 機能別 Lambda ごとに 1 ユニット（Social Shield / Motivation / Logistics / Orchestrator それぞれ独立）

---

### 質問 2: リポジトリ・ディレクトリ構成
コードをどのように整理しますか？

A) モノレポ（1 リポジトリ内に frontend / backend / infra ディレクトリ）
B) マルチリポ（frontend / backend / infra を別リポジトリ）
C) モノレポ（1 リポジトリ内に機能別ディレクトリ: social-shield / motivation / logistics 等）
X) その他（[Answer]: の後に記述してください）

[Answer]: A) モノレポ（1 リポジトリ内に frontend / backend / infra ディレクトリ）

---

### 質問 3: Lambda のコード共有方針
複数の Lambda 関数間で共通コード（DynamoDB アクセス / Bedrock 呼び出し等）をどう管理しますか？

A) Lambda Layer（共通ライブラリを Layer として共有）
B) 各 Lambda に共通コードをコピー（シンプル、依存なし）
C) 共通モジュールディレクトリ（shared/ フォルダ、デプロイ時にバンドル）
X) その他（[Answer]: の後に記述してください）

[Answer]: B) 各 Lambda に共通コードをコピー（シンプル、依存なし）

---

### 質問 4: CDK スタックの分割
AWS CDK のスタック構成はどうしますか？

A) 1 スタックにすべてのリソースをまとめる（シンプル）
B) フロントエンド用 / バックエンド用 / データ用 の 3 スタックに分割
C) 機能別スタック（Social Shield / Motivation / Logistics それぞれ独立）
X) その他（[Answer]: の後に記述してください）

[Answer]: A) 1 スタックにすべてのリソースをまとめる（シンプル）

---

回答が完了したら「回答しました」とお知らせください。
