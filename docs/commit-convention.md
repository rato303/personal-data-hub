# コミット規約

このドキュメントは、Personal Data Hubプロジェクトにおけるコミットメッセージの規約を定義します。

## 目的

- コミット履歴を読みやすくする
- 変更内容を一目で理解できるようにする
- AIエージェント（メイン・サブエージェント）とユーザーで統一されたコミットを実現

## コミットメッセージ形式

### 基本フォーマット

```
<emoji> <type>: <subject>

<body>

<footer>
```

### 絵文字 + type（必須）

変更の種類を示す絵文字とプレフィックス：

| 絵文字 | Type | 説明 | 例 |
|--------|------|------|-----|
| ✨ | `feature` | 新機能の追加 | ✨ feature: 体重データ登録APIを追加 |
| 🐛 | `fix` | バグ修正 | 🐛 fix: 体重データの小数点以下が保存されない問題を修正 |
| 📝 | `docs` | ドキュメントのみの変更 | 📝 docs: アーキテクチャ図を更新 |
| 💄 | `style` | コードの動作に影響しない変更（フォーマット、セミコロン等） | 💄 style: Kotlinコードをフォーマット |
| ♻️ | `refactor` | バグ修正や機能追加を含まないコード改善 | ♻️ refactor: 体重データ取得処理を共通化 |
| ✅ | `test` | テストの追加・修正 | ✅ test: 体重データ登録APIのテストを追加 |
| 🔧 | `chore` | ビルド、補助ツール、ライブラリ等の変更 | 🔧 chore: Spring Boot 3.2.0にアップデート |
| ⚡ | `perf` | パフォーマンス改善 | ⚡ perf: 体重データ取得クエリを最適化 |

### subject（必須）

- 変更内容を簡潔に記述（50文字以内推奨）
- 日本語または英語（プロジェクト内で統一）
- 文末にピリオド不要
- 命令形で記述（例: "追加する" ではなく "追加"）

### body（任意）

- 変更の理由や背景を詳しく説明
- subjectとbodyの間に空行を入れる
- 72文字で折り返し推奨
- 「何を」変更したかより「なぜ」変更したかを重視

### footer（任意）

- 破壊的変更（Breaking Changes）の記載
- Issue番号の参照（例: `Refs #123`, `Closes #456`）

## 言語選択

**このプロジェクトでは日本語を使用します**

理由：
- 個人プロジェクトで、主な利用者は日本語話者
- ドキュメントも日本語が中心
- 変更内容を自然に理解しやすい

## コミット粒度

### 推奨される粒度

- **1つのコミット = 1つの論理的な変更**
- レビュー可能な単位で分割
- ビルドが通る状態を保つ

### 良い例

```
✨ feature: 体重データ登録APIを追加

POST /api/weight-logs エンドポイントを実装。
バリデーション、リポジトリ層、サービス層を含む。

Refs #12
```

```
✅ test: 体重データ登録APIのテストを追加

正常系・異常系（バリデーションエラー、重複登録）のテストケースを追加。
```

### 悪い例

```
update code
```
（何を変更したか不明、絵文字・typeがない）

```
✨ feature: 体重データ登録APIを追加、テストを追加、ドキュメントを更新、リファクタリング
```
（複数の変更を1コミットにまとめすぎ）

## 特別なケース

### 初回コミット

```
🎉 chore: 初回コミット

プロジェクト構成、ドキュメント、開発環境設定を追加。
```

### マージコミット

マージコミットのメッセージはデフォルトのまま（`Merge branch 'feature/xxx'`）で問題ありません。

### Revert

```
⏪ revert: feature: 体重データ登録APIを追加

This reverts commit abc123def456.

理由: パフォーマンス問題が発覚したため一時的に取り消し
```

### その他の絵文字（任意）

基本の8種類以外にも、状況に応じて以下の絵文字を使用できます：

| 絵文字 | 用途 |
|--------|------|
| 🎉 | 初回コミット |
| ⏪ | revert（取り消し） |
| 🔀 | マージ |
| 🚀 | デプロイ関連 |
| 🔒 | セキュリティ修正 |
| ⬆️ | 依存関係のアップグレード |
| ⬇️ | 依存関係のダウングレード |
| 🗃️ | データベース関連 |

## AIエージェント向けの指示

### コミット実行前の確認事項

1. `git status` で変更内容を確認
2. `git diff` で差分を確認
3. 最近のコミット履歴（`git log --oneline -10`）でスタイルを確認
4. この規約に従ってメッセージを生成

### コミットメッセージ生成の手順

1. 変更内容を分析し、適切な絵文字と `type` を選択
2. 変更の本質を50文字以内の `subject` にまとめる
3. 必要に応じて `body` で理由や背景を説明
4. 以下のフッターを**必ず**付与：

```
🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

### コミット実行

```bash
git add <files>
git commit -m "$(cat <<'EOF'
<emoji> <type>: <subject>

<body>

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**例:**

```bash
git commit -m "$(cat <<'EOF'
📝 docs: コミット規約を追加

絵文字プレフィックスを含むコミットメッセージ規約を作成。
視覚的な分かりやすさとマシンリーダブル性を両立。

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

## 参考資料

- [Conventional Commits](https://www.conventionalcommits.org/)
- [Angular Commit Message Guidelines](https://github.com/angular/angular/blob/main/CONTRIBUTING.md#commit)
- [gitmoji](https://gitmoji.dev/) - Git commit emoji guide

## ブランチ命名規則との統一

このコミット規約は、ブランチ命名規則とも統一されています。

- ブランチ名: `feature/1`, `fix/5`, `docs/2`
- コミットメッセージ: `✨ feature:`, `🐛 fix:`, `📝 docs:`

詳細は [docs/git-workflow.md](git-workflow.md) を参照してください。

## 変更履歴

| 日付 | 変更内容 |
|------|---------|
| 2025-10-19 | 初版作成 |
| 2025-10-19 | `feat` を `feature` に変更（ブランチ命名規則と統一） |
