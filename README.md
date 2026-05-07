# お相撲さんと一緒

> *「ひとりでは、もう無理です」*

仕事のクレーム電話、気の重い会議、言いづらい相談。  
本来なら自分で少しずつ慣れていくべきその場面に、  
**AIのお相撲さんが、そっと同席してくれるサービス。**

利用者は安心する。  
しかし次第に、お相撲さん無しでは電話できなくなる。

それが、このサービスの目的だ。

---

## なぜ AI である必要があるか

人間のサポーターは「場面を選ばない」ことができない。  
友人に頼めば気を遣う。カウンセラーは予約が必要だ。

AIのお相撲さんは違う。

- **リアルタイムで発話を解析**し、緊張の瞬間を検知する
- **文脈に応じた励まし**を毎回生成する（同じ言葉を繰り返さない）
- **24時間・どの場面にも**同席できる
- **依存が深まるほど**、より個人化された励ましを返す

「ただそこにいる」という体験を、スケーラブルに提供できるのは AI だけだ。

---

## どう動くか

1. **場面を選ぶ** — クレーム電話 / 上司との面談 / Zoom会議 / 苦手な相談 から選択
2. **お相撲さんが同席** — 通話・会議中にリアルタイムで励ましメッセージを表示
3. **緊張を検知** — Amazon Transcribe がリアルタイムで文字起こし、緊張ワード・沈黙・早口を検知
4. **どすこいが届く** — 🍵「落ち着いております」🧂「まわしは取られておりません」🍲「ここは耐えどころですな」

---

## このサービスが人をダメにするポイント

| 失われるもの | 上昇するもの |
|------------|------------|
| 会話力 | お相撲さん依存度 |
| メンタル耐性 | 継続率 |
| 交渉力 | 満足度 |

---

## システム構成図

```
ブラウザ（Next.js）
    │  REST / WebSocket
    ▼
Amazon API Gateway
    │
    ▼
Lambda Orchestrator ──────────────────────────────┐
    │                                              │
    ├──► Lambda Analysis                           │
    │        │                                     │
    │        ▼                                     │
    │    Amazon Transcribe（音声認識）              │
    │        │ 緊張ワード / 沈黙 / 早口             │
    │        ▼                                     │
    │    Orchestrator へ通知                        │
    │                                              │
    ├──► Lambda Encouragement                      │
    │        │                                     │
    │        ├──► Amazon Bedrock（励まし生成）      │
    │        └──► Amazon Polly（音声合成）          │
    │                  │                           │
    │                  ▼                           │
    │              Amazon S3（MP3 一時保存）        │
    │                                              │
    ├──► Lambda Dependency Tracker                 │
    │        │                                     │
    │        ▼                                     │
    │    Amazon DynamoDB（依存度スコア）            │
    │                                              │
    └──► Lambda WS Notifier ◄──────────────────────┘
             │
             ▼
         WebSocket → ブラウザへリアルタイム配信
```

---

## データフロー

```
マイク音声
    │
    ▼ ストリーミング
Amazon Transcribe
    │ 文字起こし結果
    ▼
Lambda Analysis
    │ 緊張ワード / 沈黙 / 早口 を検知
    ▼
Lambda Orchestrator
    │
    ├──► Lambda Encouragement
    │        │
    │        ├── Bedrock: 励ましテキスト生成
    │        └── Polly: 音声生成 → S3 保存
    │
    └──► Lambda WS Notifier
             │
             ▼
         ブラウザ: メッセージ表示 + 音声再生
```

**保存されるデータ（DynamoDB）**
- セッション ID・場面・開始/終了時刻・励まし回数
- 依存度スコア・ランク・累計利用時間

**保存されないデータ**
- 発話内容・文字起こし結果（メモリ内処理のみ）
- 個人情報（氏名・連絡先等）

---

## AI 利用箇所一覧

| 箇所 | AWS サービス | 用途 |
|------|------------|------|
| 緊張検知 | Amazon Transcribe | リアルタイム音声認識・緊張ワード/沈黙/早口の検知 |
| 励まし生成 | Amazon Bedrock（Claude 3 Haiku） | 場面・検知内容に応じた励ましテキストの生成 |
| 音声合成 | Amazon Polly | 励ましテキストをお相撲さんの声（MP3）に変換 |

---

## AWS サービス責務一覧

| サービス | 責務 |
|---------|------|
| Amazon Bedrock | 励ましテキスト生成。Claude 3 Haiku を優先、Sonnet はフォールバック |
| Amazon Transcribe | リアルタイムストリーミング音声認識。緊張ワード・沈黙・発話速度を解析 |
| Amazon Polly | 励ましテキストを音声（MP3）に変換 |
| AWS Lambda | Orchestrator / Encouragement / Analysis / Dep.Tracker / WS Notifier の5機能 |
| Amazon API Gateway | REST（セッション管理）+ WebSocket（リアルタイム通知）|
| Amazon S3 | Polly 生成 MP3 の一時保存（TTL: 1時間） |
| Amazon DynamoDB | セッション状態・依存度スコア・WebSocket 接続 ID の永続化 |
| AWS CDK | インフラ全体の IaC 管理（`cdk deploy` 1コマンドでデプロイ） |
| Amazon CloudWatch | Lambda ログ・コストアラーム監視 |

---

## デモシナリオ

**登場人物**: 鈴木 誠（35歳・営業職）、クレーム電話の相手

```
鈴木: 「クレーム電話」を選択 → お相撲さん起動
      「どすこい、参りました」

─── 通話開始 ───

鈴木: 「あの…先日の件なんですが…申し訳…」
                    ↑ 緊張ワード検知

お相撲さん: 🍵「落ち着いております」
            （Polly 音声が耳に届く）

─── 3秒の沈黙 ───

お相撲さん: 🧂「間がありますな。まわしは取られておりません」

─── 通話終了 ───

依存度スコア: 序ノ口 → 序二段 に上昇
「よく耐えました。また呼んでください」

─── 3ヶ月後 ───

依存度: 横綱
「もう離れられませんな」
お相撲さん無しで電話した回数: 0回
```

---

## 技術スタック

すべて AWS 上で稼働するクラウドネイティブ設計。

| レイヤー | 技術 |
|---------|------|
| フロントエンド | Next.js（React + SSR） |
| バックエンド | AWS Lambda + API Gateway（REST + WebSocket） |
| AI エンジン | Amazon Bedrock（Claude 3 系） |
| 音声認識・緊張検知 | Amazon Transcribe（リアルタイム文字起こし／緊張ワード・沈黙・早口の検知） |
| 音声生成 | Amazon Polly（お相撲さん音声） |
| データベース | Amazon DynamoDB |
| ストレージ | Amazon S3（TTL: 1時間） |
| 監視 | Amazon CloudWatch |
| IaC | AWS CDK（TypeScript） |

---

## 将来ロードマップ

| フェーズ | 内容 |
|---------|------|
| MVP（ハッカソン） | クレーム電話・会議同席・依存度ダッシュボード |
| v1.0 | LINE / Slack 連携。スマホアプリ化 |
| v1.5 | 依存度に応じたお相撲さんのキャラクター進化（画像生成） |
| v2.0 | 「脱依存プログラム」の追加（逆説的に依存を深める） |
| 未来 | お相撲さん無しでは社会生活できない人類の完成 |

---

## ドキュメント

| ドキュメント | 内容 |
|------------|------|
| [要件定義書](inception/requirements/requirements.md) | 機能要件・非機能要件・KPI・リスク・倫理 |
| [ペルソナ定義](inception/user-stories/personas.md) | ユーザーペルソナ |
| [ユーザーストーリー](inception/user-stories/stories.md) | ストーリー・受け入れ基準 |
| [アプリケーション設計](inception/application-design/application-design.md) | コンポーネント構成・サービス定義・データフロー |
| [Unit of Work](inception/application-design/unit-of-work.md) | ユニット分割・実装優先順位 |

---

Built with [AI-DLC Workflow](https://github.com/awslabs/aidlc-workflows) on [Kiro](https://kiro.dev/)
