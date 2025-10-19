# ADR 002: AWSアーキテクチャ推奨構成

## ステータス
提案中

## 日付
2025-10-19

## コンテキスト

ADR 001で定義した要件に基づき、コスパを最重視したAWSアーキテクチャを選定する。
個人利用(1ユーザー)のため、トラフィックは低く、サーバーレス/従量課金型が適している。

## 検討した選択肢

### パターンA: フルAWSマネージドサービス

**構成**
- データベース: RDS PostgreSQL (t4g.micro)
- バックエンド: Lambda + API Gateway
- 定期実行: EventBridge + Step Functions
- フロントエンド: S3 + CloudFront
- 認証: Cognito
- 画像解析: Textract / Rekognition

**月額コスト試算: 約$25**
- RDS t4g.micro: $15
- Lambda + API Gateway: $3
- S3 + CloudFront: $2
- EventBridge + Step Functions: $2
- Textract/Rekognition: $3

**メリット**
- すべてAWSネイティブで統合管理しやすい
- RDSの自動バックアップ・スナップショット
- AWS学習に最適

**デメリット**
- RDSのコストが高め
- データベース停止時間を設けても月額$10前後

### パターンB: Supabase活用ハイブリッド

**構成**
- データベース: Supabase (Free Tier: 500MB DB, 1GB storage)
- バックエンド: Lambda + API Gateway
- 定期実行: EventBridge + Step Functions
- フロントエンド: S3 + CloudFront
- 認証: Supabase Auth (または Cognito)
- 画像解析: Textract / Rekognition

**月額コスト試算: 約$10**
- Supabase Free: $0
- Lambda + API Gateway: $3
- S3 + CloudFront: $2
- EventBridge + Step Functions: $2
- Textract/Rekognition: $3

**メリット**
- 大幅なコスト削減
- Supabaseは認証・ストレージも含む
- PostgreSQL互換でDomaも使用可能

**デメリット**
- AWS外のサービス依存
- VPC内からのアクセスに工夫が必要
- Free Tierの制限 (500MB DBは個人利用なら十分)

### パターンC: App Runner + RDS

**構成**
- データベース: RDS PostgreSQL (t4g.micro)
- バックエンド: App Runner (0.25 vCPU, 0.5GB)
- 定期実行: EventBridge + Lambda
- フロントエンド: S3 + CloudFront
- 認証: Cognito
- 画像解析: Textract / Rekognition

**月額コスト試算: 約$20-30**
- RDS: $15
- App Runner: $5-15 (トラフィック次第)
- その他: $5

**メリット**
- コンテナデプロイが簡単
- コールドスタートなし
- Spring Bootがそのまま動く

**デメリット**
- Lambdaより高コスト
- 低トラフィックではオーバースペック

## 決定

**パターンA (フルAWS) を推奨** - ただし段階的に実装

### 理由

1. **学習目的**: AWS学習が主目的の一つであり、AWSネイティブサービスの理解が深まる
2. **将来性**: 個人利用から拡張する場合もAWS内で完結
3. **統合性**: IAM、VPC、CloudWatch等の統合管理が容易
4. **コスト最適化の余地**:
   - RDS停止スケジュールで$10削減可能
   - Aurora Serverless v2への移行も検討可能

### 段階的実装プラン

**Phase 1: MVP (最小構成)**
- RDS t4g.micro (開発時は停止スケジュール設定)
- Lambda + API Gateway
- S3 + CloudFront
- 認証なし (開発用)

**Phase 2: 機能追加**
- EventBridge + Step Functions (定期実行)
- Textract (画像解析)

**Phase 3: 認証実装**
- Cognito
- ソーシャルログイン (Google, X)

**Phase 4: 本番最適化**
- CloudWatch監視
- コスト最適化レビュー

## アーキテクチャ図

```
┌─────────────┐
│   ユーザー   │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────────┐
│  CloudFront (CDN)                   │
└──────┬──────────────────────────────┘
       │
       ▼
┌──────────────┐      ┌────────────────────┐
│ S3 (React)   │      │ API Gateway        │
└──────────────┘      └─────────┬──────────┘
                                 │
                                 ▼
                      ┌──────────────────────┐
                      │ Lambda               │
                      │ (Kotlin + Spring)    │
                      └─────────┬────────────┘
                                │
                                ▼
                      ┌──────────────────────┐
                      │ RDS PostgreSQL       │
                      │ (t4g.micro)          │
                      └──────────────────────┘

┌──────────────────────────────────────────────┐
│  定期実行フロー                                │
├──────────────────────────────────────────────┤
│  EventBridge (cron)                          │
│         ↓                                    │
│  Step Functions                              │
│         ↓                                    │
│  Lambda (CSV取込/スクレイピング)              │
│         ↓                                    │
│  Textract (OCR)                              │
│         ↓                                    │
│  RDS PostgreSQL                              │
└──────────────────────────────────────────────┘

┌──────────────────────────────────────────────┐
│  認証 (Phase 3)                               │
├──────────────────────────────────────────────┤
│  Cognito User Pool                           │
│    - Google OAuth                            │
│    - X (Twitter) OAuth                       │
└──────────────────────────────────────────────┘
```

## 技術的考慮事項

### Kotlin + Spring BootのLambda実行

**課題**: コールドスタート時間 (5-10秒)

**対策**:
1. **Provisioned Concurrency** (月額+$10程度)
   - 常時ウォームなインスタンスを確保
   - 個人利用では不要

2. **GraalVM Native Image**
   - 起動時間を100-200msまで短縮
   - ビルドが複雑化
   - Spring Boot 3.0+ で公式サポート

3. **SnapStart** (Javaのみ)
   - Kotlinは未サポート

**判断**: MVP段階ではコールドスタートを許容し、Phase 4でGraalVM検討

### データベース接続管理

**課題**: Lambdaからの接続プーリング

**対策**:
- RDS Proxy使用 (月額+$10)
- または Lambda内で接続を使い回し (Spring Bootなら自動)

## 影響

- 月額約$25でフル機能のWebアプリケーションが運用可能
- AWS各種サービスの実践的な学習機会
- 将来的な拡張性を確保

## 参考資料

- [AWS Lambda with Kotlin](https://docs.aws.amazon.com/lambda/latest/dg/lambda-kotlin.html)
- [Spring Boot on AWS Lambda](https://spring.io/guides/gs/serverless/)
- [RDS Pricing Calculator](https://calculator.aws/)
