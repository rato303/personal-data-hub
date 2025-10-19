# Personal Data Hub - Gemini Code Assist Style Guide

このドキュメントは、Gemini Code AssistによるPRレビュー時に参照されるプロジェクト固有のルールとガイドラインです。

## プロジェクト概要

**Personal Data Hub** は、個人の生活データ（健康データ、カラオケ採点など）を一元管理するデータハブです。

- **利用者**: 1人（個人プロジェクト）
- **技術スタック**: Kotlin/Spring Boot（バックエンド）+ React/TypeScript（フロントエンド）+ PostgreSQL（DB）+ AWS（インフラ）
- **運用方針**: サーバーレス・マネージドサービス活用、低コスト運用（月$10-15）

## コミット規約

### 基本フォーマット

```
<emoji> <type>: <subject>

<body>

<footer>
```

### type と絵文字（8種類）

| 絵文字 | Type | 説明 |
|--------|------|------|
| ✨ | `feature` | 新機能の追加 |
| 🐛 | `fix` | バグ修正 |
| 📝 | `docs` | ドキュメントのみの変更 |
| 💄 | `style` | コードの動作に影響しない変更（フォーマット等） |
| ♻️ | `refactor` | バグ修正や機能追加を含まないコード改善 |
| ✅ | `test` | テストの追加・修正 |
| 🔧 | `chore` | ビルド、補助ツール、ライブラリ等の変更 |
| ⚡ | `perf` | パフォーマンス改善 |

### コミットメッセージの例

```
✨ feature: 体重データ登録APIを追加

POST /api/weight-logs エンドポイントを実装。
バリデーション、リポジトリ層、サービス層を含む。

Closes #12
```

**重要**:
- 日本語を使用
- subjectは50文字以内
- 1つのコミット = 1つの論理的な変更

詳細: [docs/commit-convention.md](../docs/commit-convention.md)

## ブランチ命名規則

```
<type>/<issue番号>[-<optional-description>]
```

### 例

```
feature/1
fix/5
docs/2-workflow
chore/10
```

**ポイント**:
- issue番号は必須
- descriptionは任意（issue番号があれば内容は参照可能）
- typeはコミット規約のtypeと一致

詳細: [docs/git-workflow.md](../docs/git-workflow.md)

## Pull Request ガイドライン

### PRタイトル

コミットメッセージと同じ形式を使用：

```
✨ feature: GitHub MCP設定ドキュメントを追加
```

### PR Description

```markdown
## 概要
このPRは何をするか（1-2行で簡潔に）

## 関連issue
Closes #1

## 変更内容
- 変更点1
- 変更点2

## テスト
- [ ] 手動テスト完了
- [ ] 単体テスト追加/更新
- [ ] 統合テスト追加/更新
```

**重要**:
- `Closes #<issue番号>` を必ず含める（マージ時に自動クローズ）
- 変更内容を簡潔に説明

## コーディング規約

### バックエンド（Kotlin/Spring Boot）

- **言語バージョン**: Kotlin 1.9+
- **フレームワーク**: Spring Boot 3.x
- **SQLマッパー**: Doma
- **テスト**: Spock (Groovy) + Testcontainers

**規約**:
- Kotlinの慣用句に従う
- Nullセーフティを重視
- Spring Bootのベストプラクティスに準拠

### フロントエンド（React/TypeScript）

- **言語バージョン**: TypeScript 5.x
- **フレームワーク**: React 18+
- **ビルド**: Vite
- **状態管理**: Zustand / Context API
- **テスト**: Vitest + React Testing Library

**規約**:
- 関数コンポーネントとHooksを使用
- TypeScriptの型安全性を最大限活用
- propsの型定義は必須

### データベース（PostgreSQL）

- **バージョン**: PostgreSQL 18
- **実行環境**: EC2 t4g.micro + Docker Compose

**規約**:
- テーブル名、カラム名はスネークケース
- 外部キー制約を適切に設定
- インデックスは必要な箇所のみ（個人利用、データ量少）

## アーキテクチャ原則

1. **ドキュメントファースト**: 実装前に設計書を作成
2. **テスト重視**: 単体テスト・統合テストで品質を担保
3. **個人プロジェクト**: 過度な最適化は不要
4. **コスト重視**: AWS無料枠を最大限活用

詳細: [docs/architecture.md](../docs/architecture.md)

## レビュー時の重点項目

### 必須チェック項目

- [ ] コミットメッセージが規約に従っているか
- [ ] PRタイトルとdescriptionが適切か
- [ ] `Closes #<issue番号>` が含まれているか
- [ ] テストが追加・更新されているか
- [ ] 型安全性が保たれているか（TypeScript/Kotlin）

### 推奨チェック項目

- [ ] コードの可読性（変数名、関数名）
- [ ] 適切なエラーハンドリング
- [ ] パフォーマンスへの配慮（個人利用の範囲内）
- [ ] セキュリティ上の問題（認証、バリデーション等）

### レビュー時の注意点

- **個人プロジェクト**: 完璧を求めすぎない
- **学習目的**: 新しい技術の試行は歓迎
- **コスト優先**: 高価なサービスは避ける

## 参考ドキュメント

- [アーキテクチャ](../docs/architecture.md) - システム全体像
- [開発ワークフロー](../docs/development-workflow.md) - 要件定義〜実装の流れ
- [コミット規約](../docs/commit-convention.md) - コミットメッセージ詳細
- [Gitワークフロー](../docs/git-workflow.md) - ブランチ戦略とPR手順
- [ADR](../docs/adr/) - アーキテクチャ決定記録

## Claude Code Actions との併用

このプロジェクトでは、**Claude Code Actions**（主要レビュー）と**Gemini Code Assist**（セカンドオピニオン）を併用しています。

- **Claude Code Actions**: `@claude` での手動呼び出し、詳細なレビュー
- **Gemini Code Assist**: 自動レビュー、異なる視点での指摘

両方のレビュー結果を比較し、プロジェクトに適した運用を模索中です。

---

**最終更新**: 2025-10-19
