# Personal Data Hub - 実装ノート

> 最終更新: 2025-10-19

## 概要

このドキュメントでは実装時の技術的課題と対策を記載します。

## 技術的課題と対策

### 1. Kotlin + Spring BootのLambdaコールドスタート
- **課題**: 初回起動5-10秒かかる（AWS Serverless Java Container使用）
- **対策**:
  - Phase 1では許容（個人利用のため影響小）
  - Phase 4でGraalVM Native Image検討（100-200msに短縮可能）
- **詳細**: ADR 004参照

### 2. Lambda - EC2 PostgreSQL接続管理
- **課題**: 接続プーリング
- **対策**:
  - VPC内接続（ENI経由）
  - Spring Bootの接続管理に依存
  - 必要に応じてコネクションプール設定調整

### 3. Webスクレイピング
- **課題**: カラオケサイトのログイン・認証
- **対策**:
  - Selenium/Puppeteer on Lambda
  - または別途EC2で実行

### 4. データベース運用
- **課題**: バックアップ・リストアの自動化
- **対策**:
  - CloudWatch Events で自動バックアップ
  - Ansible Playbookでリストア手順をIaC化
  - メンテナンスツール整備

### 5. テスト戦略

#### バックエンドテスト
- **Phase 1**: 単体テスト（Spock）+ 統合テスト（Testcontainers）
  - ビジネスロジック、Repository層のテスト
  - PostgreSQL使用の統合テスト
- **Phase 3以降**: E2Eテストは基本的にフロントエンド側で実施
- **詳細**: ADR 005, 007参照

#### フロントエンドテスト
- **Phase 1**: 単体テスト（Vitest + React Testing Library）
  - コンポーネントテスト
  - カバレッジ目標: 80%以上
- **Phase 3**: E2Eテスト（Playwright）導入
  - 認証フロー全体のテスト
  - 重要なユーザーフロー（ログイン、データアップロード、データ閲覧）
  - API も含めた全フローをテスト
- **詳細**: ADR 008参照

## 参考資料

- [システムアーキテクチャ](../architecture.md)
- [ADR (Architecture Decision Records)](../adr/)
- [開発ロードマップ](roadmap.md)
