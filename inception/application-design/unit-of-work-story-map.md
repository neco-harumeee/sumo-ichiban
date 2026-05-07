# ユーザーストーリー マッピング — お相撲さんと一緒

## ストーリー → ユニット マッピング

| ストーリー ID | ストーリー概要 | 主担当ユニット | 関連ユニット |
|-------------|-------------|--------------|------------|
| US-01 | 場面を選んでお相撲さんを呼ぶ | U1 Frontend | U2 Orchestrator |
| US-02 | お相撲さんの同席を終了する | U1 Frontend | U2 Orchestrator, U5 Dep.Tracker |
| US-03 | 電話中に励ましメッセージを受け取る | U4 Analysis | U2 Orchestrator, U3 Encouragement |
| US-04 | お相撲さんの声で励まされる | U3 Encouragement | U2 Orchestrator |
| US-05 | 沈黙が続いたときに助け舟を出してもらう | U4 Analysis | U3 Encouragement |
| US-06 | Zoom 会議の隅にお相撲さんを常駐させる | U1 Frontend | U2 Orchestrator |
| US-07 | 長時間ミュートを検知して励ましを受け取る | U4 Analysis | U3 Encouragement |
| US-08 | 早口・詰まりを検知して励ましを受け取る | U4 Analysis | U3 Encouragement |
| US-09 | 依存度スコアを確認する | U1 Frontend（ダッシュボード）| U5 Dep.Tracker |
| US-10 | 依存度に応じたメッセージを受け取る | U5 Dep.Tracker | U3 Encouragement |
| US-11 | 利用履歴を確認する | U1 Frontend（履歴 UI）| U5 Dep.Tracker |

## エピック → ユニット マッピング

| エピック | 主担当ユニット |
|---------|--------------|
| E-01: 場面選択・起動 | U1 Frontend + U2 Orchestrator |
| E-02: クレーム電話同行 | U4 Analysis + U3 Encouragement |
| E-03: 会議同席 | U4 Analysis + U3 Encouragement |
| E-04: 依存度の可視化 | U5 Dep.Tracker + U1 Frontend |
| 横断: リアルタイム通知 | WS Notifier（U2 内） |
| 横断: インフラ | U6 Infra |

## デモシナリオ フロー（鈴木 誠 視点）
1. [U1] Web UI で「クレーム電話」を選択 → お相撲さん起動（US-01）
2. [U2] Orchestrator がセッション開始・Analysis Lambda 起動
3. [U4] 通話中に「申し訳」を検知 → Orchestrator に通知（US-03）
4. [U3] Bedrock で励ましテキスト生成 → Polly で音声生成（US-04）
5. [U1] 励ましメッセージ「落ち着いております」と音声が画面に届く
6. [U4] 3秒の沈黙を検知 → 「間がありますな」を表示（US-05）
7. [U1] 「終了」ボタンでセッション終了（US-02）
8. [U5] 依存度スコアが上昇 → ランクが「序二段」に（US-09）
9. 数ヶ月後: 依存度「横綱」→「もう離れられませんな」（US-10）
