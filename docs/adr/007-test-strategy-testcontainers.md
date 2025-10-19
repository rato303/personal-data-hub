# ADR 007: テスト戦略 - Testcontainersの採用

## 日付
2025-10-19

## コンテキスト

バックエンドのテスト戦略を定義する必要がある。
特に、データベース（PostgreSQL）を使用する統合テストをどう実装するかが課題。

### テストの種類
1. **単体テスト**: ビジネスロジックのテスト（モックを使用）
2. **統合テスト**: データベースを含むテスト（実際のSQLを実行）

統合テストにおいて、本番環境と同じPostgreSQLを使ってテストしたい。

## 検討した選択肢

### 選択肢A: インメモリデータベース（H2）

**概要**
- H2などのインメモリデータベースを使用
- テスト実行時にメモリ上にDBを構築

**メリット**
- 起動が速い
- 設定が簡単

**デメリット**
- **本番環境との乖離**: PostgreSQL固有の機能（JSON型、全文検索など）が使えない
- **SQLの互換性**: H2とPostgreSQLで微妙に挙動が異なる
- **信頼性**: 本番で動かないコードがテストでは通る可能性

### 選択肢B: 共有データベース

**概要**
- 開発環境に共有のPostgreSQLを用意
- テストはその共有DBに対して実行

**メリット**
- 本番環境と同じPostgreSQL

**デメリット**
- **テスト間の干渉**: 複数のテストが同時実行されると干渉する
- **クリーンアップが困難**: テストデータのクリーンアップが必要
- **CI/CDとの統合が困難**: 各開発者ごとに環境が異なる

### 選択肢C: Testcontainers

**概要**
- Dockerコンテナを使用して、テストごとに使い捨てのPostgreSQLを起動
- テスト終了後にコンテナを破棄

**メリット**
- **本番環境と同じPostgreSQL**: 完全な互換性
- **テスト間の独立性**: 各テストが独立したDBを使用
- **クリーンアップ不要**: コンテナごと破棄されるため、データが残らない
- **CI/CDとの統合が容易**: Docker環境があればどこでも動作

**デメリット**
- Dockerが必要
- 起動時間がインメモリDBより遅い（ただし数秒程度）

## 決定

**選択肢C: Testcontainers** を採用する。

## 理由

### 1. 本番環境との完全な互換性
PostgreSQL 18をDockerコンテナとして起動するため、本番環境と完全に同じ環境でテストできる。
PostgreSQL固有の機能（JSON型、配列型、全文検索など）を安心して使える。

### 2. テスト間の独立性確保
各テストクラス（または各テストメソッド）ごとに新しいコンテナを起動できるため、テスト間でデータが干渉しない。

```groovy
@Testcontainers
class UserRepositorySpec extends Specification {
    @Container
    static PostgreSQLContainer postgres = new PostgreSQLContainer("postgres:18")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test")

    def "ユーザーを保存できる"() {
        given: "ユーザーエンティティ"
        def user = new User(name: "Alice")

        when: "保存する"
        repository.save(user)

        then: "データベースに保存されている"
        repository.findById(user.id).isPresent()
    }
}
```

### 3. クリーンアップ不要
テスト終了後にコンテナごと破棄されるため、手動でのデータクリーンアップが不要。
常にクリーンな状態でテストが実行される。

### 4. CI/CDとの統合が容易
GitHub ActionsやCircleCIなどのCI環境にもDocker環境があれば、ローカルと同じようにテストが実行できる。

```yaml
# GitHub Actions の例
- name: Run tests
  run: ./gradlew test
  # Testcontainersが自動的にDockerコンテナを起動
```

### 5. 開発者間での環境統一
各開発者のローカル環境に違いがあっても、Dockerコンテナで統一された環境でテストできる。

## テスト戦略

### 単体テスト（Unit Test）
- **対象**: ビジネスロジック、バリデーション、ユーティリティクラス
- **モック**: Spockの組み込みモック機能を使用
- **実行速度**: 高速

```groovy
def "バリデーション - メールアドレス形式チェック"() {
    expect:
    validator.isValidEmail(email) == isValid

    where:
    email               || isValid
    "test@example.com"  || true
    "invalid"           || false
}
```

### 統合テスト（Integration Test）
- **対象**: Repository層、データベースを使う処理
- **データベース**: Testcontainers（PostgreSQL 18）
- **実行速度**: 中速（コンテナ起動のオーバーヘッドあり）

```groovy
@Testcontainers
class BloodPressureRepositorySpec extends Specification {
    @Container
    static PostgreSQLContainer postgres = new PostgreSQLContainer("postgres:18")

    def "血圧データを保存して取得できる"() {
        given: "血圧データ"
        def data = new BloodPressureData(systolic: 120, diastolic: 80)

        when: "保存する"
        repository.save(data)

        then: "取得できる"
        def found = repository.findById(data.id)
        found.isPresent()
        found.get().systolic == 120
    }
}
```

### E2Eテスト
- **対象**: API全体のフロー
- **実行**: 必要に応じて（Phase 1では優先度低）

## 影響

### ポジティブ
- 本番環境との完全な互換性
- テストの信頼性向上
- テスト間の干渉がない
- CI/CD環境での実行が容易

### ネガティブ
- Docker環境が必要
- テスト実行時間がわずかに長くなる（コンテナ起動）

### 対策
- **コンテナの再利用**: `@Container` を `static` にして、テストクラス全体で1つのコンテナを共有
- **並列実行**: Gradleの並列テスト実行機能を活用

```kotlin
// build.gradle.kts
tasks.test {
    maxParallelForks = Runtime.getRuntime().availableProcessors()
}
```

## 運用

### ローカル環境
- Docker Desktop / Docker Engine が必要
- `./gradlew test` でテスト実行（Testcontainersが自動的にコンテナ起動）

### CI/CD環境
- GitHub Actions、CircleCIなどでDocker環境を有効化
- Testcontainersが自動的に動作

```yaml
# .github/workflows/test.yml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-java@v3
      - run: ./gradlew test
```

## 関連する決定
- ADR 003: データベースをRDSからEC2 + Docker PostgreSQLへ変更
- ADR 005: テストフレームワークとしてSpockを採用

## 参考資料
- [Testcontainers Documentation](https://www.testcontainers.org/)
- [Testcontainers for Java](https://www.testcontainers.org/modules/databases/postgres/)
- [Spock + Testcontainers](https://www.testcontainers.org/test_framework_integration/spock/)
