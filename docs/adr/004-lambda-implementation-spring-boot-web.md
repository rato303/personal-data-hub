# ADR 004: Lambda実装方式としてSpring Boot Webを採用

## 日付
2025-10-19

## コンテキスト

Kotlin + Spring Boot を AWS Lambda で実行する方法には、複数のアプローチが存在する。
個人プロジェクトとして、学習効率とコストのバランスを考慮した実装方式を選定する必要がある。

### 学習優先度（再確認）
- **高優先**: Step Functions, Lambda, SQS, CloudFront, Cognito
- **低優先**: RDS、Lambda特化の最適化技術

### 要件
- Kotlin + Spring Boot + Doma の組み合わせ
- ローカル開発がしやすいこと
- 既存の Spring Boot 知識を活用できること
- MVP段階ではコールドスタートを許容

## 検討した選択肢

### 選択肢A: Spring Cloud Function

**概要**
- AWS Lambda 用に最適化された Spring の軽量フレームワーク
- 関数型プログラミングモデル
- 複数のクラウドプロバイダ対応

**実装例**
```kotlin
@Bean
fun uppercase(): (String) -> String {
    return { it.uppercase() }
}
```

**コールドスタート時間**: 2-3秒

**メリット**
- Lambda に最適化されている
- 起動が比較的高速
- 関数型で設計がシンプル
- AWS以外のクラウドにも対応

**デメリット**
- 通常の Spring Boot Web (`@RestController`) とは異なるプログラミングモデル
- Doma との統合に工夫が必要
- 学習コストが増加（今回の学習優先度と異なる）
- 一般的な Spring Boot スキルとしての汎用性が低い

### 選択肢B: Spring Boot Web + AWS Serverless Java Container（推奨）

**概要**
- 通常の Spring Boot Web アプリケーションをそのまま Lambda で実行
- AWS Serverless Java Container がリクエストを変換
- `@RestController`, `@Service` などがそのまま使える

**実装例**
```kotlin
@RestController
@RequestMapping("/api")
class HealthDataController(
    private val healthDataRepository: HealthDataRepository
) {
    @GetMapping("/blood-pressure")
    fun getBloodPressure(): List<BloodPressure> {
        return healthDataRepository.findAll()
    }
}
```

**コールドスタート時間**: 5-10秒

**メリット**
- 一般的な Spring Boot の知識がそのまま使える
- Doma との統合が容易（通常の Spring Boot と同じ）
- ローカル開発がしやすい（通常の Spring Boot アプリとして起動可能）
- 既存のSpring Boot リソース・ノウハウが活用できる
- 一般的なスキルとして汎用性が高い

**デメリット**
- コールドスタートが遅い（5-10秒）
- Lambda特化の最適化ではない

### 選択肢C: Spring Boot Web + GraalVM Native Image

**概要**
- Spring Boot アプリをネイティブバイナリにコンパイル
- Spring Boot 3.0+ で公式サポート

**コールドスタート時間**: 100-200ms

**メリット**
- コールドスタート非常に高速
- メモリ使用量削減
- Spring Boot Web の知識がそのまま使える

**デメリット**
- ビルドが非常に複雑
- ビルド時間が長い（数分）
- リフレクション・プロキシの制約
- Doma との互換性検証が必要
- MVP段階では過剰最適化

## 決定

**選択肢B（Spring Boot Web + AWS Serverless Java Container）を採用**

### 理由

1. **学習優先度との整合性**
   - Lambda特化の最適化（Spring Cloud Function, GraalVM）は学習優先度が低い
   - 本プロジェクトの学習目標は Step Functions, SQS, CloudFront, Cognito
   - Spring Boot Web は一般的なスキルとして汎用性が高い

2. **開発効率**
   - 通常の Spring Boot Web の知識がそのまま使える
   - ローカル開発が容易（Spring Boot アプリとして起動）
   - Doma との統合が標準的な方法で実現可能

3. **MVP段階の割り切り**
   - 個人利用（1人）のためコールドスタート5-10秒は許容範囲
   - 初回アクセス以外は高速（Lambda コンテナ再利用）
   - 機能実装・学習を優先

4. **将来の最適化パス**
   - Phase 4 で GraalVM Native Image への移行を検討可能
   - Spring Boot Web をベースにしているため、移行パスが明確
   - 必要に応じて Provisioned Concurrency も選択肢

## 実装詳細

### 依存関係

**build.gradle.kts**
```kotlin
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("com.amazonaws.serverless:aws-serverless-java-container-springboot3:2.0.0")
    implementation("org.seasar.doma.boot:doma-spring-boot-starter:3.0.0")
    // その他
}
```

### Lambda Handler

```kotlin
class StreamLambdaHandler : SpringBootLambdaContainerHandler<AwsProxyRequest, AwsProxyResponse>() {
    init {
        initialize(Application::class.java)
    }
}
```

### デプロイ設定

**Lambda設定**
- **Runtime**: Java 17
- **Memory**: 512MB（必要に応じて調整）
- **Timeout**: 30秒
- **Environment Variables**: Spring Profile等

## 影響

### ポジティブ
- 開発速度の向上（既存知識の活用）
- Doma統合が容易
- ローカル開発環境の整備が簡単
- 一般的なスキルとして蓄積

### ネガティブ
- コールドスタート時間が長い（5-10秒）
  - ただし個人利用のため影響は限定的
  - Phase 4 で GraalVM 検討

### リスクと対策
- **リスク**: コールドスタートによるユーザー体験悪化
  - **対策**: MVP段階では許容、Phase 4 で GraalVM 検討
- **リスク**: Lambda実行時間による課金増加
  - **対策**: 個人利用（低トラフィック）のため影響小、必要に応じて最適化

## 今後の検討事項（Phase 4）

### GraalVM Native Image への移行
- Spring Boot 3.0+ の Native Image サポート
- Doma との互換性検証
- ビルドパイプラインの整備
- コールドスタート 100-200ms 達成

### Provisioned Concurrency
- コールドスタート回避のため常時ウォームなインスタンスを確保
- コスト: 月額+$10程度
- 個人利用では不要だが、選択肢として認識

## 参考資料

- [AWS Serverless Java Container](https://github.com/awslabs/aws-serverless-java-container)
- [Spring Boot on AWS Lambda](https://spring.io/guides/gs/serverless/)
- [Spring Cloud Function](https://spring.io/projects/spring-cloud-function)
- [Spring Boot GraalVM Native Image](https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html)
