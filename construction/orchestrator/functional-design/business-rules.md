# ビジネスルール — Unit 2: Orchestrator

## BR-01: セッション重複起動の自動解決
- **ルール**: 同一 user_id で ACTIVE なセッションが存在する場合、既存セッションを自動的に CANCELLED に更新してから新規セッションを開始する
- **根拠**: デモ操作の利便性を優先（エラーで止まらない）
- **例外**: なし

## BR-02: 条件付き Lambda 起動
- **ルール**: セッション開始時、`enabled_features` の各フラグが `true` の場合のみ対応する Lambda を起動する
- **デフォルト**: 全フラグ `true`（全部盛り）
- **起動対象**:
  - `social_shield_enabled` → Social Shield Lambda
  - `logistics_enabled` → Logistics Lambda
  - `motivation_rebuttal_enabled` → EventBridge Scheduler 登録

## BR-03: モチベ反論スケジュール
- **ルール**: `motivation_rebuttal_enabled == true` の場合、`end_time_planned` ちょうどに EventBridge Scheduler で Motivation Lambda をトリガーする
- **1 回限り**: 繰り返しトリガーは行わない
- **キャンセル**: セッション終了時にスケジュールをキャンセルしない（シンプル化のため）
  - ※ セッション終了後に Motivation Lambda が起動した場合、Lambda 側でセッション状態を確認して処理をスキップする

## BR-04: セッション終了処理のシンプル化
- **ルール**: セッション終了時は状態更新とサマリー生成のみ行う。EventBridge スケジュールのキャンセルや他 Lambda への通知は行わない
- **根拠**: ハッカソン PoC のため実装をシンプルに保つ

## BR-05: 終了時刻バリデーション
- **ルール**: `end_time_planned` は現在時刻より未来でなければならない
- **エラー**: 過去の時刻が指定された場合は `400 Bad Request` を返す

## BR-06: セッション ID の一意性
- **ルール**: `session_id` は UUID v4 で生成し、一意性を保証する

## BR-07: アクション記録の追記
- **ルール**: `actions_taken` は追記のみ（削除・更新不可）
- **目的**: デモ時のサマリー表示の正確性を保証する

## バリデーションルール

| 入力フィールド | バリデーション |
|-------------|-------------|
| `user_id` | 必須、空文字不可 |
| `end_time_planned` | 必須、ISO8601 形式、現在時刻より未来 |
| `session_id`（終了時） | 必須、存在するセッションの ID |
| `action_type` | ActionType 列挙値のいずれか |
