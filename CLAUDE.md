# Claude用プロジェクトガイド

このドキュメントは、Claude（AIアシスタント）がこのプロジェクトを理解し、効率的に作業するためのガイドです。

## プロジェクト概要

**プロジェクト名**: Personal Data Hub

**概要**: 個人の生活データ（健康データ、カラオケ採点など）を一元管理するデータハブ。様々なデータソースから情報を取り込み、可視化・分析するWebアプリケーション。

**利用者**: 1人（個人プロジェクト）

**目的**:
- 日々のデータ入力・管理の手間を削減（CSV取込、OCR、スクレイピングの自動化）
- 散在する生活データを一元管理し、長期的な傾向分析を可能にする
- サーバーレス・マネージドサービスを活用した低コスト運用（月$10-15）

## ドキュメント構成

### 最初に読むべきドキュメント

1. **[docs/architecture.md](docs/architecture.md)** - システムアーキテクチャ全体
   - システム構成図（Mermaid）
   - 技術スタック（フロントエンド・バックエンド）
   - データベース設計の概要
   - インフラ構成（AWS）

2. **[docs/directory-structure.md](docs/directory-structure.md)** - ディレクトリ構成
   - プロジェクト全体の構造
   - 各ディレクトリの役割

3. **[docs/development-workflow.md](docs/development-workflow.md)** - 開発ワークフロー
   - 要件定義 → 設計 → 実装 → テストの流れ
   - ドキュメントテンプレートへのリンク

4. **[docs/git-workflow.md](docs/git-workflow.md)** - Gitワークフロー
   - ブランチ命名規則（`feature/1`, `fix/5`等）
   - issueベースの開発フロー
   - PRの作成とマージ手順

### 設計判断の記録（ADR）

**場所**: [docs/adr/](docs/adr/)

**読み方**: 必要な時に参照する（最初から全部読む必要はない）

### プロジェクト管理

**場所**: [docs/plans/](docs/plans/)

- [roadmap.md](docs/plans/roadmap.md): 開発ロードマップ（Phase 1〜4）
- [implementation-notes.md](docs/plans/implementation-notes.md): 技術的課題と対策
- [next-actions.md](docs/plans/next-actions.md): 次のアクション（直近TODO）

### テンプレート

**場所**: [docs/templates/](docs/templates/)

#### 要件定義用
- [requirements/usecase-template.md](docs/templates/requirements/usecase-template.md)

#### 設計用
- [design/api-spec-template.md](docs/templates/design/api-spec-template.md)
- [design/behavior-spec-template.md](docs/templates/design/behavior-spec-template.md)
- [design/frontend-spec-template.md](docs/templates/design/frontend-spec-template.md)
- [design/batch-spec-template.md](docs/templates/design/batch-spec-template.md)
- [design/workflow-spec-template.md](docs/templates/design/workflow-spec-template.md)
- [design/usecase-index-template.md](docs/templates/design/usecase-index-template.md)

#### ADR用
- [adr/adr-template.md](docs/templates/adr/adr-template.md)

## 技術スタック

**概要**: Kotlin/Spring Boot（バックエンド）+ React/TypeScript（フロントエンド）+ PostgreSQL（DB）+ AWS（インフラ）

**詳細**: [docs/architecture.md](docs/architecture.md) の「技術スタック」セクションを参照

## Claudeへの指示

### 新しいユースケースを実装する場合

1. 既存の業務フロー（`docs/requirements/bf-*.md`）を確認
2. 該当するユースケースがなければ、テンプレートから作成
3. `docs/design/usecase-index.md` にユースケースを追加
4. 種別に応じたディレクトリ（api/batch/workflow/webhook）に設計書を作成
5. テンプレートを活用してドキュメント作成
6. 実装前に設計書の確認を依頼

### 技術選定や設計判断をする場合

1. **必ず [docs/how-to-write-adr.md](docs/how-to-write-adr.md) を参照**
2. 既存のADR（`docs/adr/`）を確認（同様の判断がないか）
3. 新しい判断の場合、テンプレート（[docs/templates/adr/adr-template.md](docs/templates/adr/adr-template.md)）からADRを作成
4. ADRは作成後、内容を変更しない（不変）
5. 決定を変更する場合は、新しいADRを作成し、古いADRを `docs/adr/superseded/` に移動

### issueベースで開発する場合

1. **必ず [docs/git-workflow.md](docs/git-workflow.md) を参照**
2. issueを確認してブランチを作成（`<type>/<issue番号>`）
3. 作業完了後、PRを作成
4. PR descriptionに `Closes #<issue番号>` を記載
5. マージ後、ブランチを削除

### コミットを作成する場合

1. **必ず [docs/commit-convention.md](docs/commit-convention.md) を参照**
2. 変更内容を `git status` と `git diff` で確認
3. 最近のコミット履歴で既存のスタイルを確認
4. 規約に従ってコミットメッセージを生成
5. 必ずClaudeのフッター（Co-Authored-By）を含める

### ドキュメント参照の優先順位

1. **architecture.md**（全体像把握）- 必ず最初に読む
2. **development-workflow.md**（作業手順）- 必ず読む
3. **ADR**（設計判断の理由）- 疑問が生じた時に参照
4. **templates/**（具体的な書き方）- ドキュメント作成時に参照

## 参考リンク

- [既存IaC構成（instance-provisioning）](https://github.com/rato303/instance-provisioning)

## 注意事項

- **個人プロジェクト**: 利用者1人、過度な最適化は不要
- **コスト重視**: AWS無料枠を最大限活用、月$10-15で運用
- **ドキュメントファースト**: 実装前に設計書を作成
- **テスト重視**: 単体テスト・統合テストで品質を担保
