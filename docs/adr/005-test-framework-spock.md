# ADR 005: テストフレームワークとしてSpockを採用

## 日付
2025-10-19

## コンテキスト

バックエンド（Kotlin + Spring Boot）のテストフレームワークを選定する必要がある。
振る舞い仕様書（behavior-spec.md）とテストコードを1対1対応させ、Given-When-Then形式でテストを記述したい。

## 検討した選択肢

### 選択肢A: JUnit 5 + Mockito

**概要**
- JVM系の標準的なテストフレームワーク
- Mockitoでモック・スタブを実現

**メリット**
- 広く使われており、情報が豊富
- Kotlinとの親和性が高い
- IDEサポートが充実

**デメリット**
- Given-When-Thenの構文が強制されない（アノテーションで@DisplayNameなどで表現）
- モックライブラリ（Mockito）を別途追加する必要がある
- テストコードの記述が冗長になりがち

### 選択肢B: Spock

**概要**
- Groovyベースのテストフレームワーク
- Given-When-Then構文が言語レベルでサポート
- モック・スタブ機能が組み込み

**メリット**
- **Given-When-Then構文の強制**: 振る舞い仕様書と1対1対応しやすい
- **モックライブラリ不要**: 組み込みのモック機能が充実
- **表現力が高い**: Groovyの動的な表現力でテストが読みやすい
- **データテーブル**: パラメタライズドテストが直感的に書ける
- **アサーションメッセージ**: 失敗時のメッセージが自動的に詳細

**デメリット**
- Groovyの学習コストがわずかにある
- Kotlinとの混在（実装はKotlin、テストはGroovy）

### 選択肢C: Kotest

**概要**
- Kotlin専用のテストフレームワーク
- BDD形式のテストもサポート

**メリット**
- Kotlin Nativeで書ける
- 様々なテストスタイルをサポート

**デメリット**
- Spockほどの歴史がない
- Given-When-Thenの強制力が弱い
- 振る舞い仕様書との対応が明確でない

## 決定

**選択肢B: Spock** を採用する。

## 理由

### 1. Given-When-Then構文の強制
Spockはテストコードの構造として Given-When-Then ブロックを言語レベルで強制する。
これにより、振る舞い仕様書（behavior-spec.md）とテストコードが自然に1対1対応する。

```groovy
def "CSVアップロード - 正常系"() {
    given: "有効なCSVファイルが準備されている"
    def file = new MockMultipartFile("file", "data.csv", "text/csv", validCsvData)

    when: "APIを呼び出す"
    def response = api.uploadCsv(file)

    then: "ステータスコード200が返却される"
    response.statusCode == 200
    response.body.status == "success"
}
```

### 2. モックライブラリ不要
Mockitoなどの外部ライブラリを追加せずに、組み込みのモック・スタブ機能が使える。

```groovy
given: "リポジトリがモックされている"
def repository = Mock(UserRepository)
repository.findById(1) >> Optional.of(user)
```

### 3. データテーブルによるパラメタライズドテスト
複数のテストケースを表形式で簡潔に記述できる。

```groovy
def "バリデーション - 異常系"() {
    expect:
    validator.validate(input).hasErrors() == hasError

    where:
    input       || hasError
    ""          || true
    null        || true
    "valid"     || false
}
```

### 4. 振る舞い仕様書との整合性
開発ワークフローとして、behavior-spec.md → Spockテストコード という流れが確立できる。

## 影響

### ポジティブ
- 振る舞い仕様書とテストコードの1対1対応が自然に実現
- テストコードの可読性向上
- モック・スタブの記述が簡潔

### ネガティブ
- チームメンバーがGroovyを学ぶ必要がある（ただし基本的な構文のみ）
- Kotlinとの混在（実装とテストで異なる言語）

### 対策
- Spockの基本的な書き方をドキュメント化
- テンプレート（behavior-spec-template.md）を用意して学習コストを削減

## 関連する決定
- ADR 004: Lambda実装方式としてSpring Boot Webを採用
- ADR 007: テスト戦略 - Testcontainersの採用

## 参考資料
- [Spock Framework Documentation](https://spockframework.org/spock/docs/2.3/index.html)
- [Spock Primer](https://spockframework.org/spock/docs/2.3/spock_primer.html)
