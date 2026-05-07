# Application Design プラン — お相撲さんと一緒

## 実行チェックリスト（全完了）
- [x] Step 1: コンポーネント定義（components.md）
- [x] Step 2: コンポーネントメソッド定義（component-methods.md）
- [x] Step 3: サービス層設計（services.md）
- [x] Step 4: コンポーネント依存関係（component-dependency.md）
- [x] Step 5: 統合ドキュメント（application-design.md）

## 設計方針の回答
- フロントエンド: Next.js（React + SSR）
- バックエンド: 機能ごとに Lambda を分割（Orchestrator / Encouragement / Analysis / Dep.Tracker / WS Notifier）
- AI エンジン: Bedrock 直接呼び出し（Agents なし。励まし生成はシンプルなプロンプトで十分）
- 音声認識: Amazon Transcribe（リアルタイムストリーミング）
- 音声生成: Amazon Polly → S3
- セッション状態管理: DynamoDB（永続化）
- 通信方式: REST + WebSocket 併用
- インフラ管理: AWS CDK（TypeScript）
