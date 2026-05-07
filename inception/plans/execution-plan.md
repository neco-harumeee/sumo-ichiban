# 実行計画 — お相撲さんと一緒

## 計画サマリー

| 項目 | 内容 |
|------|------|
| **プロジェクト** | お相撲さんと一緒 |
| **タイプ** | Greenfield |
| **想定ユニット数** | 6 |
| **リスク** | Medium |
| **期間** | ハッカソン期間内 |

---

## ステージ実行計画

| ステージ | 実行判定 | 理由 |
|---------|---------|------|
| Workspace Detection | EXECUTE | 必須 |
| Reverse Engineering | SKIP | Greenfield |
| Requirements Analysis | EXECUTE | 必須 |
| User Stories | EXECUTE | デモ映えに必要 |
| Workflow Planning | EXECUTE | 必須 |
| Application Design | EXECUTE | 必須 |
| Units Generation | EXECUTE | 必須 |
| Functional Design | EXECUTE | ユニット毎 |
| NFR Requirements | EXECUTE | ユニット毎 |
| NFR Design | EXECUTE | ユニット毎 |
| Infrastructure Design | EXECUTE | ユニット毎 |
| Code Generation | EXECUTE | ユニット毎 |
| Build and Test | EXECUTE | 必須 |
| Operations | PLACEHOLDER | ハッカソン後 |

---

## 想定ユニット構成

| ID | ユニット | 優先度 | 依存 |
|----|---------|--------|------|
| U-01 | Frontend（Next.js） | 高 | U-02 |
| U-02 | Orchestrator Lambda | 高 | U-03, U-04, U-05 |
| U-03 | Encouragement Lambda（Bedrock + Polly） | 高 | なし |
| U-04 | Analysis Lambda（Transcribe + Rekognition） | 高 | なし |
| U-05 | Dependency Tracker Lambda（DynamoDB） | 中 | なし |
| U-06 | Infrastructure（AWS CDK） | 高 | 全ユニット |

---

## リスク

| リスク | 影響 | 対策 |
|--------|------|------|
| Rekognition のリアルタイム表情分析の遅延 | Medium | デモではモック実装で代替 |
| Transcribe のリアルタイム音声認識の精度 | Medium | 緊張ワードリストで補完 |
| Bedrock の応答速度 | Low | キャッシュ・プリセットメッセージで補完 |
| ハッカソン期間内の実装量 | High | コア機能（FR-01〜FR-03）を優先、FR-04〜FR-06 はモック |
