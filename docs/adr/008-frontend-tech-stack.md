# ADR 008: フロントエンド技術スタックの選定

## ステータス
承認済み

## 日付
2025-10-19

## コンテキスト

フロントエンド（React + TypeScript）の技術スタックを選定する必要がある。
特に、ビルドツール、テストフレームワーク、状態管理ライブラリの選定が重要。

バックエンドと同様に、テスト戦略を明確にし、品質を担保できる構成にしたい。

## 決定事項

以下の技術スタックを採用する：

### ビルドツール: Vite
### テストフレームワーク（単体）: Vitest + React Testing Library
### テストフレームワーク（E2E）: Playwright（Phase 3以降）
### 状態管理: Zustand / Context API
### リンター: ESLint + Prettier

## 選定理由

### 1. ビルドツール: Vite

**比較対象: Create React App (CRA)**

#### Viteを選ぶ理由
- **圧倒的な速度**: ESビルドベースで、開発サーバー起動が爆速（CRAの10倍以上速い）
- **HMR（Hot Module Replacement）**: 変更が即座に反映
- **モダン**: 2020年リリース、現代的なツールチェーン
- **TypeScript完全サポート**: 設定不要
- **本番ビルド**: Rollupベースで最適化されたバンドル

#### CRAのデメリット
- 起動が遅い（Webpackベース）
- メンテナンスが停滞気味
- カスタマイズにejectが必要

**結論**: Viteを採用

---

### 2. テストフレームワーク（単体）: Vitest + React Testing Library

**比較対象: Jest + React Testing Library**

#### Vitestを選ぶ理由
- **Viteとの統合**: 設定ファイルを共有、一貫性
- **Jest互換**: Jestのほぼすべての機能が使える、移行コストゼロ
- **高速**: Viteの恩恵で爆速テスト実行
- **TypeScript完全サポート**: 型安全なテスト

#### React Testing Libraryを選ぶ理由
- **ユーザー視点のテスト**: 実装詳細に依存しない
- **ベストプラクティス**: Reactコミュニティのスタンダード
- **アクセシビリティ重視**: アクセシブルなUIを自然に書ける

```typescript
// React Testing Library の例
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'

test('CSVアップロードフォーム - 正常系', async () => {
  render(<CsvUploadForm />)

  const fileInput = screen.getByLabelText('CSVファイルを選択')
  const file = new File(['data'], 'test.csv', { type: 'text/csv' })

  await userEvent.upload(fileInput, file)
  await userEvent.click(screen.getByRole('button', { name: 'アップロード' }))

  expect(await screen.findByText('アップロード成功')).toBeInTheDocument()
})
```

**結論**: Vitest + React Testing Libraryを採用

---

### 3. テストフレームワーク（E2E）: Playwright（Phase 3以降）

**比較対象: Cypress**

#### Playwrightを選ぶ理由
- **モダン**: Microsoft製、2020年リリース
- **高速**: Cypressより速い
- **並列実行**: 複数ブラウザで並列テスト
- **TypeScript完全サポート**: 型安全なテスト
- **複数ブラウザ対応**: Chromium、Firefox、WebKit
- **APIテスト**: ブラウザなしでAPIテストも可能

```typescript
// Playwright の例
test('ログインフロー', async ({ page }) => {
  await page.goto('https://app.example.com')
  await page.fill('input[name="email"]', 'test@example.com')
  await page.fill('input[name="password"]', 'password')
  await page.click('button[type="submit"]')

  await expect(page).toHaveURL(/.*dashboard/)
})
```

#### Cypressのデメリット
- やや古い（2014年リリース）
- 並列実行が有料プラン

**結論**: Playwright を Phase 3（認証導入時）から採用

---

### 4. 状態管理: Zustand / Context API

**比較対象: Redux、Recoil**

#### Zustandを選ぶ理由（複雑な状態管理が必要な場合）
- **シンプル**: Reduxより圧倒的に簡潔
- **Boilerplate不要**: アクション、リデューサーの定義が不要
- **TypeScript完全サポート**: 型推論が効く
- **パフォーマンス**: 不要な再レンダリングを回避

```typescript
// Zustand の例
import create from 'zustand'

interface UserState {
  user: User | null
  setUser: (user: User) => void
}

const useUserStore = create<UserState>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
}))
```

#### Context APIを選ぶ理由（シンプルな状態管理の場合）
- **標準機能**: 追加ライブラリ不要
- **学習コスト低い**: React開発者なら誰でも知っている

**方針**:
- Phase 1（認証なし）: Context APIで十分
- Phase 3（認証あり）: 必要に応じてZustandに移行

**結論**: Context API → 必要に応じてZustand

---

### 5. リンター: ESLint + Prettier

#### ESLintを選ぶ理由
- **デファクトスタンダード**: JavaScriptのリンター
- **TypeScript対応**: @typescript-eslint で完全対応
- **プラグイン**: React、Hooksのベストプラクティスを強制

#### Prettierを選ぶ理由
- **コードフォーマッター**: コードスタイルの統一
- **チーム開発**: レビューでのスタイル議論を排除

**結論**: ESLint + Prettier を採用

---

## テスト戦略

### Phase 1: 単体テスト中心
- **Vitest + React Testing Library**: コンポーネントテスト
- **カバレッジ目標**: 80%以上

### Phase 3: E2Eテスト導入
- **Playwright**: 認証フロー全体のテスト
- **重要なユーザーフロー**: ログイン、データアップロード、データ閲覧

## 影響

### ポジティブ
- 開発体験の向上（Viteの高速ビルド）
- テストの信頼性向上（実環境に近いテスト）
- バックエンドと同等のテスト品質
- TypeScript完全対応で型安全

### ネガティブ
- 新しいツールの学習コスト（ただし、ほぼJest/CRA互換）

### 対策
- Vite、Vitestの基本的な使い方をドキュメント化
- React Testing Libraryのベストプラクティスを共有

## 関連する決定
- ADR 005: テストフレームワークとしてSpockを採用（バックエンド）
- ADR 007: テスト戦略 - Testcontainersの採用（バックエンド）

## 参考資料
- [Vite Documentation](https://vitejs.dev/)
- [Vitest Documentation](https://vitest.dev/)
- [React Testing Library](https://testing-library.com/react)
- [Playwright Documentation](https://playwright.dev/)
- [Zustand Documentation](https://github.com/pmndrs/zustand)
