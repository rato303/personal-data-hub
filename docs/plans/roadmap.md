# Personal Data Hub - 開発ロードマップ

> 最終更新: 2025-10-19

## 概要

このドキュメントではPersonal Data Hubの開発計画を記載します。

## 開発フェーズ

### Phase 1: MVP (認証なし) - 現在
- [x] プロジェクト構成決定
- [x] ADR作成（ADR 001, 002, 003）
- [ ] IaC構築（Pulumi + Ansible）
- [ ] EC2 + PostgreSQL セットアップ
- [ ] バックエンド基盤構築 (Kotlin + Spring Boot + Doma)
- [ ] フロントエンド基盤構築 (React + TypeScript)
- [ ] Lambda デプロイ
- [ ] S3 + CloudFront デプロイ
- [ ] CSV取込機能 (血圧・体組成)

### Phase 2: 画像解析・定期実行
- [ ] EventBridge + Step Functions セットアップ
- [ ] SQS統合
- [ ] Textract統合 (健康診断画像OCR)
- [ ] Webスクレイピング (カラオケサイト)

### Phase 3: 認証
- [ ] Cognito User Pool作成
- [ ] Google OAuth設定
- [ ] X (Twitter) OAuth設定
- [ ] フロントエンド認証フロー実装

### Phase 4: 本番最適化
- [ ] CloudWatch監視設定
- [ ] コスト最適化レビュー
- [ ] GraalVM Native Image検討 (コールドスタート改善)

## 参考資料

- [システムアーキテクチャ](../architecture.md)
- [ADR (Architecture Decision Records)](../adr/)
