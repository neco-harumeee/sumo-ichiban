# ユーザーストーリー マッピング — TRAINING LOCK

## ストーリー → ユニット マッピング

| ストーリー ID | ストーリー概要 | 主担当ユニット | 関連ユニット |
|-------------|-------------|--------------|------------|
| US-01 | トレーニング開始宣言 | U1 Frontend | U2 Orchestrator |
| US-02 | トレーニング終了宣言 | U1 Frontend | U2 Orchestrator |
| US-03 | 仕事メールへの自動返信 | U3 Social Shield | U2 Orchestrator, U6 WS Notifier |
| US-04 | メール返信文のカスタマイズ | U1 Frontend（設定 UI）| U3 Social Shield |
| US-05 | 家族への自動連絡 | U3 Social Shield | U2 Orchestrator, U6 WS Notifier |
| US-06 | 連絡先・テンプレートの事前登録 | U1 Frontend（設定 UI）| U2 Orchestrator |
| US-07 | サボり検知とモチベ反論の生成 | U4 Motivation Rebuttal | U2 Orchestrator, U6 WS Notifier |
| US-08 | モチベ反論の音声送信 | U4 Motivation Rebuttal | U6 WS Notifier |
| US-09 | 反論の圧の強さ設定 | U1 Frontend（設定 UI）| U4 Motivation Rebuttal |
| US-10 | トレーニングギアの自動発注 | U5 Logistics | U2 Orchestrator, U6 WS Notifier |
| US-11 | 発注履歴の確認 | U1 Frontend（履歴 UI）| U5 Logistics |

## ユニット別ストーリー集計

| ユニット | 主担当ストーリー | 関連ストーリー | 合計 |
|---------|---------------|-------------|------|
| U1 Frontend | US-01, US-02, US-04, US-06, US-09, US-11 | — | 6 |
| U2 Orchestrator | — | US-01〜US-11（全体制御） | — |
| U3 Social Shield | US-03, US-05 | US-04, US-06 | 2 |
| U4 Motivation Rebuttal | US-07, US-08 | US-09 | 2 |
| U5 Logistics | US-10 | US-11 | 1 |
| U6 WS Notifier | — | US-03, US-05, US-07, US-08, US-10 | — |
| U7 Infra | — | 全ユニットのリソース定義 | — |

## エピック → ユニット マッピング

| エピック | 主担当ユニット |
|---------|--------------|
| エピック 1: トレーニング宣言 | U1 Frontend + U2 Orchestrator |
| エピック 2: 社会的シールド | U3 Social Shield |
| エピック 3: モチベ反論機能 | U4 Motivation Rebuttal |
| エピック 4: Amazon 物流連携 | U5 Logistics |
| 横断: リアルタイム通知 | U6 WS Notifier |
| 横断: インフラ | U7 Infra |

## デモシナリオ フロー（田中 剛 視点）

```
1. [U1] Web UI でトレーニング開始宣言（US-01）
        ↓
2. [U2] Orchestrator がセッション開始・各 Lambda 起動
        ↓
3. [U3] 仕事メールに自動返信（US-03）→ [U6] WS 通知
   [U5] ギア自動発注（US-10）→ [U6] WS 通知
        ↓
4. [U4] 終了時刻超過 → サボり検知 → 反論生成 → Polly 音声 → 送信（US-07, US-08）
        ↓
5. [U1] Web UI でトレーニング終了宣言（US-02）→ サマリー表示
```
