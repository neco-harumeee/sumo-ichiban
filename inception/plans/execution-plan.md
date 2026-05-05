# 実行計画 — TRAINING LOCK

## 詳細分析サマリー

### 変更影響評価
| 項目 | 内容 |
|------|------|
| **ユーザー向け変更** | Yes — Web UI（トレーニング宣言・設定画面） |
| **構造的変更** | Yes — 新規サービス（Greenfield）、複数 AWS サービス連携 |
| **データモデル変更** | Yes — セッション状態・連絡先・発注履歴の管理 |
| **API 変更** | Yes — API Gateway + Lambda エンドポイント新規作成 |
| **NFR 影響** | Yes — Bedrock レイテンシ・Polly 音声生成・S3 コスト |

### リスク評価
| 項目 | 評価 |
|------|------|
| **リスクレベル** | Medium |
| **ロールバック複雑度** | Moderate（AWS リソース削除で対応可） |
| **テスト複雑度** | Moderate（モック実装で工数抑制） |
| **主なリスク** | Bedrock API レイテンシ / PA-API 利用制限 / ハッカソン期間制約 |

---

## ワークフロー可視化

```
[ユーザーリクエスト]
        |
        v
+------------------------------------------+
|  INCEPTION PHASE                         |
+------------------------------------------+
| [x] Workspace Detection    COMPLETED     |
| [x] Requirements Analysis  COMPLETED     |
| [x] User Stories           COMPLETED     |
| [x] Workflow Planning      IN PROGRESS   |
| [ ] Application Design     EXECUTE       |
| [ ] Units Generation       EXECUTE       |
+------------------------------------------+
        |
        v
+------------------------------------------+
|  CONSTRUCTION PHASE (per unit)           |
+------------------------------------------+
| [ ] Functional Design      EXECUTE       |
| [ ] NFR Requirements       EXECUTE       |
| [ ] NFR Design             EXECUTE       |
| [ ] Infrastructure Design  EXECUTE       |
| [ ] Code Generation        EXECUTE       |
| [ ] Build and Test         EXECUTE       |
+------------------------------------------+
        |
        v
+------------------------------------------+
|  OPERATIONS PHASE                        |
+------------------------------------------+
| [ ] Operations             PLACEHOLDER   |
+------------------------------------------+
```

---

## 実行フェーズ詳細

### 🔵 INCEPTION PHASE

| ステージ | 判定 | 理由 |
|---------|------|------|
| Workspace Detection | ✅ COMPLETED | 実施済み |
| Reverse Engineering | ⏭ SKIPPED | Greenfield のため不要 |
| Requirements Analysis | ✅ COMPLETED | 実施済み |
| User Stories | ✅ COMPLETED | 実施済み（11 ストーリー / 4 エピック） |
| Workflow Planning | ✅ IN PROGRESS | 本ステージ |
| **Application Design** | 🟠 **EXECUTE** | 新規コンポーネント（Web UI / Lambda / Bedrock Agent / Polly / S3 / DynamoDB）の設計が必要 |
| **Units Generation** | 🟠 **EXECUTE** | 4 エピックを並行開発可能な単位に分解する価値あり |

### 🟢 CONSTRUCTION PHASE（各ユニット）

| ステージ | 判定 | 理由 |
|---------|------|------|
| **Functional Design** | 🟠 **EXECUTE** | Bedrock プロンプト設計・状態管理ロジック・メール生成ロジックの詳細設計が必要 |
| **NFR Requirements** | 🟠 **EXECUTE** | Bedrock レイテンシ・Polly 音声生成コスト・S3 ストレージ・DynamoDB スループットの要件定義が必要 |
| **NFR Design** | 🟠 **EXECUTE** | NFR パターン（非同期処理・エラーハンドリング・タイムアウト）の組み込みが必要 |
| **Infrastructure Design** | 🟠 **EXECUTE** | Lambda / API Gateway / S3 / DynamoDB / Bedrock / Polly の具体的なマッピングが必要 |
| **Code Generation** | ✅ **EXECUTE** | 常に実行（ALWAYS） |
| **Build and Test** | ✅ **EXECUTE** | 常に実行（ALWAYS） |

### 🟡 OPERATIONS PHASE

| ステージ | 判定 | 理由 |
|---------|------|------|
| Operations | ⏸ PLACEHOLDER | 将来の拡張用 |

---

## 想定ユニット構成（Units Generation で確定）

| ユニット候補 | 対応エピック | 主要 AWS サービス |
|------------|------------|----------------|
| Unit 1: Web UI + API | US-01, US-02, US-06, US-11 | API Gateway, Lambda, DynamoDB |
| Unit 2: 社会的シールド | US-03, US-04, US-05 | Bedrock, Lambda, SES（モック） |
| Unit 3: モチベ反論 | US-07, US-08, US-09 | Bedrock, Polly, S3, SES |
| Unit 4: 物流連携 | US-10 | Bedrock, Lambda, PA-API（モック） |

---

## 成功基準
- **主目標**: ハッカソンデモ動画で全機能フローが確認できる
- **主要成果物**: Web アプリ + Lambda 関数群 + Bedrock 連携 + Polly 音声生成
- **品質ゲート**: 各ユニットの動作確認 → 統合テスト → デモ動画撮影可能な状態
