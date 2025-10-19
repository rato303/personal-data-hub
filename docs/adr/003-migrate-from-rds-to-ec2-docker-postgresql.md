# ADR 003: データベースをRDSからEC2 + Docker PostgreSQLへ変更

## 日付
2025-10-19

## コンテキスト

ADR 002でRDS PostgreSQLを選定したが、以下の前提条件が明確になった：

### 学習優先度の再評価
- **高優先**: Step Functions, Lambda, SQS, CloudFront, Cognito
- **低優先**: RDS（学習は副業が安定してから）

### 利用状況
- 利用者は1人のみ
- ダウンタイム許容可能
- 高可用性・自動フェイルオーバー不要

### 運用体制
- IaC体制が整っている（Pulumi + Ansible）
- 既存の汎用開発環境構築の知見あり（`/home/ubuntu/workspace/instance-provisioning`）
- バックアップ・リストア手順のIaC化が可能

### コスト制約
- 個人プロジェクトのため、可能な限りコストを抑えたい
- AWS無料枠を最大限活用したい

## 検討した選択肢

### 選択肢A: RDS PostgreSQL (現行)

**構成**
- RDS t4g.micro
- Multi-AZ: なし
- 自動バックアップ: 有効

**月額コスト: $15**

**メリット**
- マネージドサービスで運用負荷が低い
- 自動バックアップ・PITR
- AWS学習になる

**デメリット**
- 学習優先度が低い
- 月額$15のコストが高い
- 1人利用には過剰スペック

### 選択肢B: EC2 t4g.micro + Docker PostgreSQL（推奨）

**構成**
- EC2 t4g.micro（無料枠対象）
- Docker Compose で PostgreSQL 16
- CloudWatch Events で自動バックアップ（pg_dump → S3）
- EBS Snapshot（週次）

**月額コスト: $0-5**
- EC2 t4g.micro: 無料枠12ヶ月 → その後$5/月
- EBS 20GB: 無料枠30GBに含まれる
- S3バックアップ: 数MB程度（$0）

**メリット**
- **コスト削減**: 月$10-15の節約
- **IaC体制**: 既存の知見を活用可能
- **学習優先度**: 浮いたコストでCognito等に集中できる
- **柔軟性**: PostgreSQL以外のミドルウェアも同居可能（Redis等）

**デメリット**
- 運用負荷が若干増加（バックアップスクリプトの管理）
- マネージドサービスの学習機会を失う

### 選択肢C: Supabase Free Tier

**月額コスト: $0**

**メリット**
- 完全無料
- 認証・ストレージも統合

**デメリット**
- AWS外のサービス依存（AWS学習の目的から外れる）
- VPC内からのアクセスに工夫が必要
- Free Tier制限（500MB DB、1GB storage）

## 決定

**選択肢B（EC2 + Docker PostgreSQL）を採用**

### 理由

1. **学習優先度との整合性**
   - RDSの学習は後回し
   - 浮いたコストでCognito、Step Functions等に集中

2. **コスト最適化**
   - 月$10-15の削減（年間$120-180）
   - AWS無料枠を最大限活用

3. **運用実現性**
   - 既存のIaC知見（Pulumi + Ansible）を活用
   - `/home/ubuntu/workspace/instance-provisioning` の構成を流用可能
   - バックアップ・リストア手順もIaC化

4. **利用状況に適合**
   - 1人利用に最適化
   - ダウンタイム許容（開発環境でも問題なし）

5. **将来性**
   - 必要に応じてRDSへの移行は容易（PostgreSQL互換）
   - 副業が安定したタイミングでRDSを再検討

## 実装詳細

### アーキテクチャ

```
[Lambda (Kotlin/Spring)]
    ↓ (VPC内接続)
[EC2 t4g.micro]
    ├─ Docker PostgreSQL 16
    └─ Backup Script (Cron)
        ↓
    [S3] ← 日次 pg_dump

[CloudWatch Events]
    ↓
[EBS Snapshot] ← 週次スナップショット
```

### バックアップ戦略

| 種類 | 頻度 | 保存先 | 保持期間 | RPO | RTO |
|------|------|--------|---------|-----|-----|
| pg_dump | 日次 (深夜3時) | S3 | 30日 | 24時間 | 1時間 |
| EBS Snapshot | 週次 (日曜3時) | EBS | 4世代 | 7日 | 30分 |

### IaC構成

```
health-routine-tracker/iac/
├── pulumi/
│   ├── network/        # VPC, Subnet, SecurityGroup
│   ├── database/       # EC2 + Docker PostgreSQL
│   └── tools/
│       └── maintenance/
│           ├── db-backup/   # バックアップツール
│           └── db-restore/  # リストアツール
└── ansible/
    └── roles/
        ├── postgresql/      # PostgreSQL + Docker構成
        └── backup-cron/     # バックアップCron設定
```

### セキュリティ

- SecurityGroup: Lambda用SGからのみ5432ポート許可
- パスワード: Secrets Managerで管理
- バックアップS3: サーバーサイド暗号化（SSE-S3）

## 影響

### ポジティブ
- 月額コスト: $25 → $10-15（$10-15削減）
- 学習リソース: Cognito、Step Functions等に集中可能
- 運用柔軟性: Dockerで他ミドルウェアも追加可能

### ネガティブ
- RDSの学習機会を失う（副業安定後に再検討）
- バックアップスクリプトの保守が必要

### リスクと対策
- **リスク**: EC2障害時のダウンタイム
  - **対策**: EBS Snapshotで30分以内に復旧可能
- **リスク**: バックアップ失敗
  - **対策**: CloudWatch Alarms でバックアップ失敗を通知

## 移行計画

### Phase 1: IaC準備（今回）
1. Pulumi: EC2 + VPC + SecurityGroup
2. Ansible: PostgreSQL + Backup Scripts
3. メンテナンスツール作成

### Phase 2: 開発環境デプロイ
1. dev環境にデプロイ
2. バックアップ・リストアのテスト
3. Lambda接続テスト

### Phase 3: 本番環境デプロイ（MVP後）
1. prod環境にデプロイ
2. モニタリング設定
3. バックアップ運用開始

## 参考資料

- [既存IaC構成](/home/ubuntu/workspace/instance-provisioning)
- [PostgreSQL on Docker Best Practices](https://hub.docker.com/_/postgres)
- [AWS EC2 Pricing](https://aws.amazon.com/ec2/pricing/)
- [pg_dump Documentation](https://www.postgresql.org/docs/current/app-pgdump.html)

## 今後の検討事項

- RDS学習は副業が安定したタイミングで再検討
- Aurora Serverless v2も将来的な選択肢
- Read Replicaが必要になった場合の対応
