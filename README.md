# お相撲さんと一緒

> *「ひとりでは、もう無理です」*

仕事のクレーム電話、気の重い会議、言いづらい相談。  
本来なら自分で少しずつ慣れていくべきその場面に、  
**AIのお相撲さんが、そっと同席してくれるサービス。**

利用者は安心する。  
しかし次第に、お相撲さん無しでは電話できなくなる。

それが、このサービスの目的だ。

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
| ストレージ | Amazon S3 |
| IaC | AWS CDK（TypeScript） |

---

## ドキュメント

| ドキュメント | 内容 |
|------------|------|
| [要件定義書](inception/requirements/requirements.md) | 機能要件・非機能要件 |
| [ペルソナ定義](inception/user-stories/personas.md) | ユーザーペルソナ |
| [ユーザーストーリー](inception/user-stories/stories.md) | ストーリー・受け入れ基準 |
| [アプリケーション設計](inception/application-design/application-design.md) | コンポーネント構成・サービス定義 |
| [Unit of Work](inception/application-design/unit-of-work.md) | ユニット分割・実装優先順位 |

---

Built with [AI-DLC Workflow](https://github.com/awslabs/aidlc-workflows) on [Kiro](https://kiro.dev/)
