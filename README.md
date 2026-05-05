# Peak Condition, Zero Life Skills

> *体は最高、生活能力はゼロ。*

アスリートの最大の敵は、怪我でも天気でもない。  
**日常生活**である。——メール、家事、付き合い。

**pcz（Peak Condition, Zero life skills）** は、それらをすべて AWS 上の AI エージェントに丸投げする。  
受信トレイは勝手に返信される。家族には自動で連絡が届く。ギアは自動で注文される。  
サボろうとすると、Bedrock が音声で反論してくる。

体は人生最高の仕上がりになる。  
人間としての機能は、静かに失われていく。

それが、このサービスの目的だ。

---

## どう動くか

1. **宣言する** — Web アプリで「3時間消える」と入力し、終了予定時刻をセット
2. **シールド展開** — Bedrock Agent が仕事メールに自動返信、家族に連絡を送信
3. **ギア発注** — Amazon PA-API がトレーニングギアを自動注文し「シューズ、ぽちっとしておきました」と報告
4. **反論される** — 終了時刻を過ぎてもサボっていると、Polly が圧強めの音声メッセージを送りつける

---

## こんな人に

| | |
|--|--|
| 🏃 ランナー | 2時間消えても世界に気づかれたくない人 |
| 🏋️ 筋トレ民 | セット中に未読メールが気になって集中できない人 |
| 🚴 すべてのアスリート | 鍛えることに全振りして、大人としての責任を AI に任せたい人 |

---

## 技術スタック

すべて AWS 上で稼働するクラウドネイティブ設計。

| レイヤー | 技術 |
|---------|------|
| フロントエンド | Next.js（React + SSR） |
| バックエンド | AWS Lambda + API Gateway（REST + WebSocket） |
| AI エンジン | Amazon Bedrock Agents（Claude 3 Sonnet / Haiku） |
| 音声生成 | Amazon Polly |
| データベース | Amazon DynamoDB |
| ストレージ | Amazon S3 |
| 物流連携 | Amazon PA-API（モック） |
| メール送信 | Amazon SES（モック） |
| IaC | AWS CDK（TypeScript） |

---

## ドキュメント

| ドキュメント | 内容 |
|------------|------|
| [要件定義書](aidlc-docs/inception/requirements/requirements.md) | 機能要件・非機能要件・ユーザーシナリオ |
| [ペルソナ定義](aidlc-docs/inception/user-stories/personas.md) | トレーニング愛好家「田中 剛」 |
| [ユーザーストーリー](aidlc-docs/inception/user-stories/stories.md) | 11 ストーリー / 4 エピック・受け入れ基準 |
| [アプリケーション設計](aidlc-docs/inception/application-design/application-design.md) | コンポーネント構成・サービス定義・データフロー |
| [Unit of Work](aidlc-docs/inception/application-design/unit-of-work.md) | 7 ユニット分割・モノレポ構成・実装優先順位 |

---

Built with [AI-DLC Workflow](https://github.com/awslabs/aidlc-workflows) on [Kiro](https://kiro.dev/)
