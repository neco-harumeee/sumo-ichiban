# Application Design プラン — TRAINING LOCK

## 実行チェックリスト

- [x] Step 1: コンポーネント定義（components.md）
- [x] Step 2: コンポーネントメソッド定義（component-methods.md）
- [x] Step 3: サービス層設計（services.md）
- [x] Step 4: コンポーネント依存関係（component-dependency.md）
- [x] Step 5: 統合ドキュメント（application-design.md）

---

## 質問 — アプリケーション設計方針の確認

以下の質問に回答してください。各 `[Answer]:` タグの後に選択肢の文字を記入してください。

---

### 質問 1: フロントエンドの技術スタック
Web UI はどの技術で実装しますか？

A) HTML + CSS + Vanilla JS（シンプル、依存なし）
B) React（SPA、コンポーネント指向）
C) Vue.js（軽量 SPA）
D) Next.js（React + SSR）
X) その他（[Answer]: の後に記述してください）

[Answer]: D) Next.js（React + SSR）

---

### 質問 2: バックエンドの構成
Lambda 関数の分割方針はどうしますか？

A) 機能ごとに Lambda を分割（社会的シールド用 / モチベ反論用 / 物流連携用）
B) 1つの Lambda で全機能を処理（モノリシック Lambda）
C) Bedrock Agent を中心に、Lambda はオーケストレーターのみ
X) その他（[Answer]: の後に記述してください）

[Answer]: A) 機能ごとに Lambda を分割（社会的シールド用 / モチベ反論用 / 物流連携用）

---

### 質問 3: Bedrock の利用パターン
Amazon Bedrock をどのように使いますか？

A) Bedrock Agents（エージェント機能でアクションを自動判断）
B) Bedrock InvokeModel（直接 API 呼び出し、プロンプトで制御）
C) Bedrock Converse API（会話形式で制御）
X) その他（[Answer]: の後に記述してください）

[Answer]: A) Bedrock Agents（エージェント機能でアクションを自動判断）

---

### 質問 4: セッション状態の管理
トレーニングセッションの状態（開始時刻・終了予定・実行済みアクション）をどこで管理しますか？

A) DynamoDB（永続化、セッション間で保持）
B) Lambda のメモリ内のみ（デモ用、再起動でリセット）
C) S3 に JSON ファイルとして保存
X) その他（[Answer]: の後に記述してください）

[Answer]: A) DynamoDB（永続化、セッション間で保持）

---

### 質問 5: フロントエンドとバックエンドの通信
Web UI から Lambda への通信方式はどうしますか？

A) REST API（API Gateway + Lambda、シンプル）
B) WebSocket（API Gateway WebSocket、リアルタイム更新）
C) REST + WebSocket 併用（初期操作は REST、状態更新は WebSocket）
X) その他（[Answer]: の後に記述してください）

[Answer]: C) REST + WebSocket 併用（初期操作は REST、状態更新は WebSocket）

---

### 質問 6: インフラ管理
AWS リソースの管理方法はどうしますか？

A) AWS CDK（TypeScript / Python でインフラをコード化）
B) AWS SAM（Serverless Application Model）
C) Terraform
D) マネジメントコンソールで手動作成（ハッカソン用）
X) その他（[Answer]: の後に記述してください）

[Answer]: A) AWS CDK（TypeScript / Python でインフラをコード化）

---

回答が完了したら「回答しました」とお知らせください。
