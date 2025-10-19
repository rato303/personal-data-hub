# ADR 009: コミットtypeでfeatの代わりにfeatureを使用

## 日付
2025-10-19

## コンテキスト

Personal Data Hubプロジェクトでは、issueベースの開発フローを採用しており、ブランチ命名規則とコミットメッセージの規約を統一する必要がある。

[Conventional Commits](https://www.conventionalcommits.org/) では、新機能追加のコミットtypeとして `feat` を使用することが標準とされている。一方、このプロジェクトのブランチ命名規則では `feature/<issue番号>` という形式を採用している。

2つの規約で異なる表現（`feat` vs `feature`）を使うと、開発者（現時点では1人だが、将来的には複数人の可能性もある）の認知的負荷が高まる。特に、ブランチ作成時とコミット時で異なる単語を使い分ける必要があり、一貫性が失われる。

PR #4のレビューで、この一貫性の問題が指摘され、`feat` の代わりに `feature` を使用することが推奨された。

## 決定事項

コミットメッセージのtypeとして `feature` を使用する（`feat` は使用しない）。

## 選定理由

### 検討した選択肢

1. **`feat` を使用（Conventional Commits標準）**
   - Conventional Commitsの標準に準拠
   - 自動CHANGELOG生成ツールとの互換性が高い
   - コミュニティで広く使われている

2. **`feature` を使用（ブランチ命名規則と統一）**
   - ブランチ名 `feature/<issue番号>` と完全一致
   - プロジェクト内での一貫性が高い
   - より明示的で可読性が高い

### 選定した理由

**`feature` を採用する理由：**

1. **ブランチ命名規則との完全な統一**
   - ブランチ名: `feature/1`
   - コミットメッセージ: `✨ feature: 新機能を追加`
   - 2つの規約で同じ単語を使うことで、認知的負荷を軽減

2. **可読性の向上**
   - `feature` の方が `feat` より明示的
   - 省略形を避けることで、意味が明確

3. **個人プロジェクトの特性**
   - 現時点では1人開発
   - ツール互換性よりも、開発体験と一貫性を優先
   - 必要に応じてツールの設定でカスタマイズ可能

4. **ブランチ→コミットの流れがスムーズ**
   ```bash
   # ブランチ作成
   git checkout -b feature/12

   # コミット
   git commit -m "✨ feature: 体重データ登録APIを追加"

   # 同じ単語を使うため、混乱がない
   ```

### `feat` を選ばなかった理由

- Conventional Commitsの標準だが、個人プロジェクトでは厳密に従う必要性は低い
- 自動CHANGELOG生成ツールは、設定ファイルでカスタマイズ可能
- ブランチ名との不一致による混乱の方がデメリットが大きい

## 影響

### ポジティブな影響

- **開発体験の向上**: ブランチ作成時とコミット時で同じ単語を使える
- **一貫性の向上**: プロジェクト全体で統一された用語を使用
- **可読性の向上**: `feature` の方が意味が明確
- **認知的負荷の軽減**: 2つの規約を覚える必要がない

### ネガティブな影響

- **ツール互換性の問題**: 一部の自動CHANGELOG生成ツールは `feat` を期待する可能性がある
- **Conventional Commitsからの逸脱**: 標準的な規約から外れる

### 対策

- **ツールの設定でカスタマイズ**: 多くのツール（[conventional-changelog](https://github.com/conventional-changelog/conventional-changelog)等）は設定ファイルで `feature` を `feat` と同等に扱うことが可能
- **ドキュメント化**: この決定をADRとして記録し、理由を明確にする
- **将来的な見直し**: チーム開発に移行する際、必要に応じて再検討

## 関連ドキュメント

- [コミット規約](../commit-convention.md)
- [Gitワークフロー](../git-workflow.md)

## 参考資料

- [Conventional Commits](https://www.conventionalcommits.org/)
- [PR #4 レビューコメント](https://github.com/rato303/personal-data-hub/pull/4#issuecomment-3419338880)
- [conventional-changelog Configuration](https://github.com/conventional-changelog/conventional-changelog-config-spec)

## メモ

- 将来的に自動CHANGELOG生成ツールを導入する際は、設定ファイルで `feature` を `feat` と同等に扱うよう設定する
- 他の開発者が参加する場合は、この決定を再検討する可能性がある
