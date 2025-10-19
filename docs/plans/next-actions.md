# Personal Data Hub - 次のアクション

> 最終更新: 2025-10-19

## 概要

このドキュメントでは直近の開発タスクを記載します。

## 直近のタスク

### 1. IaC構築（Pulumi + Ansible）
- 既存の `/home/ubuntu/workspace/instance-provisioning` を参考
- network, database, tools ディレクトリ作成
- PostgreSQL + バックアップスクリプトのAnsible role作成

### 2. 開発環境デプロイ
- dev環境にEC2 + PostgreSQLをデプロイ
- バックアップ・リストアのテスト

### 3. バックエンド開発
- Kotlin + Spring Boot + Domaのセットアップ
- Lambda用のビルド設定

## 未確定事項（今後の壁打ち対象）
- データベーススキーマ設計
- Lambda のパッケージング方法
- CI/CD パイプライン構成

## 参考資料

- [システムアーキテクチャ](../architecture.md)
- [開発ロードマップ](roadmap.md)
- [実装ノート](implementation-notes.md)
