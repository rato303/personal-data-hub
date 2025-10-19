# ADR 006: ビルドツールとしてGradleを採用

## 日付
2025-10-19

## コンテキスト

バックエンド（Kotlin + Spring Boot）のビルドツールを選定する必要がある。
JVM系プロジェクトのビルドツールとして、Gradle と Maven が主な選択肢となる。

## 検討した選択肢

### 選択肢A: Maven

**概要**
- JVM系の伝統的なビルドツール
- XML形式の設定ファイル（pom.xml）

**メリット**
- 長い歴史があり、広く使われている
- 情報が豊富
- IDEサポートが充実

**デメリット**
- **XML形式**: 冗長で読みにくい、記述が煩雑
- 柔軟性に欠ける（カスタムタスクの記述が難しい）
- ビルドスクリプトが長くなりがち
- やや古い印象

**pom.xml の例**
```xml
<project>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>3.2.0</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.jetbrains.kotlin</groupId>
                <artifactId>kotlin-maven-plugin</artifactId>
                <version>1.9.20</version>
            </plugin>
        </plugins>
    </build>
</project>
```

### 選択肢B: Gradle

**概要**
- JVM系の現代的なビルドツール
- Kotlin DSL / Groovy DSL で記述
- 宣言的かつ柔軟な設定が可能

**メリット**
- **Kotlin DSL**: Kotlinプロジェクトとの親和性が高い
- **簡潔な記述**: XMLよりも読みやすく、記述量が少ない
- **柔軟性**: カスタムタスクが書きやすい
- **パフォーマンス**: インクリメンタルビルド、ビルドキャッシュに対応
- **現代的**: JVM系プロジェクトのスタンダード

**デメリット**
- 学習コストがMavenより若干高い（ただしKotlin開発者には親和性が高い）

**build.gradle.kts の例**
```kotlin
plugins {
    kotlin("jvm") version "1.9.20"
    kotlin("plugin.spring") version "1.9.20"
    id("org.springframework.boot") version "3.2.0"
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    testImplementation("org.spockframework:spock-core:2.3-groovy-4.0")
}
```

## 決定

**選択肢B: Gradle（Kotlin DSL）** を採用する。

## 理由

### 1. JVM系プロジェクトのスタンダード
Gradleは現代のJVM系プロジェクトで広く使われており、Spring Boot公式でも推奨されている。
歴史はあるがMavenはやや古い印象がある。

### 2. Kotlin DSLで簡潔に記載可能
ビルドスクリプト自体をKotlinで記述できるため、型安全性が高く、IDEの補完が効く。
XMLと比較して圧倒的に読みやすく、記述量が少ない。

**比較例:**

**Maven (pom.xml)**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.2.0</version>
</dependency>
```

**Gradle (build.gradle.kts)**
```kotlin
implementation("org.springframework.boot:spring-boot-starter-web:3.2.0")
```

### 3. 柔軟性とカスタマイズ性
Gradleはタスクの定義やカスタマイズが容易。
Lambda向けのビルド（Fat JAR作成、最適化など）もスクリプトで柔軟に記述できる。

```kotlin
tasks.register<Zip>("packageLambda") {
    from(tasks.bootJar)
    archiveFileName.set("lambda-function.zip")
}
```

### 4. パフォーマンス
- インクリメンタルビルド: 変更があった部分のみビルド
- ビルドキャッシュ: 過去のビルド結果を再利用
- 並列実行: タスクの並列実行が可能

### 5. Kotlinプロジェクトとの親和性
KotlinプロジェクトでビルドスクリプトもKotlinで書けるため、統一感がある。

## 影響

### ポジティブ
- ビルドスクリプトの可読性・保守性向上
- Kotlinとの統一感
- 柔軟なビルドカスタマイズが可能
- ビルド速度の向上（インクリメンタルビルド、キャッシュ）

### ネガティブ
- Maven経験者にとっては学習コストがわずかにある
- Gradleの仕組みを理解する必要がある

### 対策
- Gradleの基本的な使い方をドキュメント化
- Spring Boot公式のGradleサンプルを参考にする

## 関連する決定
- ADR 004: Lambda実装方式としてSpring Boot Webを採用
- ADR 005: テストフレームワークとしてSpockを採用

## 参考資料
- [Gradle Documentation](https://docs.gradle.org/)
- [Gradle Kotlin DSL Primer](https://docs.gradle.org/current/userguide/kotlin_dsl.html)
- [Spring Boot with Gradle](https://spring.io/guides/gs/gradle/)
