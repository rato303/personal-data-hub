# Personal Data Hub - ディレクトリ構成

> 最終更新: 2025-10-19

## 概要

このドキュメントではPersonal Data Hubプロジェクトのディレクトリ構成を説明します。

## ディレクトリ構成

```
./
├── docs/                         # ドキュメント
│   ├── architecture.md          # 確定したアーキテクチャ
│   ├── directory-structure.md   # このファイル
│   ├── development-workflow.md  # 開発ワークフロー
│   ├── templates/               # ドキュメントテンプレート集
│   │   ├── requirements/       # 要件定義用テンプレート
│   │   │   └── usecase-template.md
│   │   └── design/             # 設計用テンプレート
│   │       ├── api-spec-template.md
│   │       ├── behavior-spec-template.md
│   │       ├── frontend-spec-template.md
│   │       ├── batch-spec-template.md
│   │       ├── workflow-spec-template.md
│   │       └── usecase-index-template.md
│   ├── requirements/            # 要件定義
│   │   ├── overview.md         # 全体概要・業務フロー
│   │   └── uc-XXX-*.md         # ユースケース（機能ごと）
│   ├── design/                  # 設計書
│   │   ├── usecase-index.md    # ユースケース一覧（インデックス）
│   │   ├── database/           # DB設計（共通）
│   │   │   ├── schema.md      # スキーマ全体
│   │   │   └── er-diagram.md  # ER図
│   │   ├── api/                # API系ユースケース
│   │   │   └── uc-XXX-*/
│   │   ├── batch/              # バッチ系ユースケース
│   │   │   └── uc-XXX-*/
│   │   ├── workflow/           # ワークフロー系ユースケース
│   │   │   └── uc-XXX-*/
│   │   └── webhook/            # Webhook系ユースケース
│   │       └── uc-XXX-*/
│   ├── adr/                     # Architecture Decision Records
│   └── plans/                   # プロジェクト管理・計画
│       ├── roadmap.md           # 開発ロードマップ（Phase別）
│       ├── implementation-notes.md  # 技術的課題と対策
│       └── next-actions.md      # 次のアクション（直近TODO）
├── iac/                          # Infrastructure as Code
│   ├── pulumi/                  # Pulumi (プロビジョニング)
│   │   ├── network/             # VPC, Subnet, SecurityGroup
│   │   ├── database/            # EC2 + Docker PostgreSQL
│   │   ├── lambda/              # Lambda + API Gateway
│   │   └── tools/
│   │       └── maintenance/     # メンテナンスツール
│   └── ansible/                 # Ansible (構成管理)
│       ├── provision.yml
│       ├── group_vars/
│       └── roles/
│           ├── postgresql/      # PostgreSQL + Docker
│           └── backup-cron/     # バックアップCron
├── backend/                      # Kotlin + Spring Boot (未作成)
└── frontend/                     # React + TypeScript (未作成)
```

## ディレクトリ詳細

### `docs/`
プロジェクトのドキュメント類を格納。

- **architecture.md**: システムアーキテクチャの確定版
- **directory-structure.md**: このファイル（ディレクトリ構成説明）
- **development-workflow.md**: 開発ワークフロー（要件→設計→実装→テストの流れ）
- **templates/**: ドキュメントテンプレート集
  - **requirements/**: 要件定義用テンプレート
    - **usecase-template.md**: ユースケース記述テンプレート
  - **design/**: 設計用テンプレート
    - **api-spec-template.md**: API仕様書テンプレート
    - **behavior-spec-template.md**: 振る舞い仕様書テンプレート（Spock用）
    - **frontend-spec-template.md**: フロントエンド仕様書テンプレート
    - **batch-spec-template.md**: バッチ仕様書テンプレート
    - **workflow-spec-template.md**: ワークフロー仕様書テンプレート
    - **usecase-index-template.md**: ユースケースインデックステンプレート
- **requirements/**: 要件定義
  - **overview.md**: 全体概要・業務フロー全体図
  - **uc-XXX-*.md**: ユースケースごとの詳細要件（機能ごとに1ファイル）
- **design/**: 設計書
  - **usecase-index.md**: ユースケース一覧インデックス（UC番号順・種別ごとの一覧）
  - **database/**: DB設計（共通）
    - **schema.md**: スキーマ全体
    - **er-diagram.md**: ER図
  - **api/**: API系ユースケース（REST API、画面操作）
    - **uc-XXX-*/**: ユースケースごとの設計
      - `api-spec.md`: API仕様
      - `behavior-spec.md`: 振る舞い仕様（Spock用）
      - `frontend-spec.md`: フロントエンド仕様（画面ありの場合）
  - **batch/**: バッチ系ユースケース（定期実行）
    - **uc-XXX-*/**: ユースケースごとの設計
      - `batch-spec.md`: バッチ仕様
      - `behavior-spec.md`: 振る舞い仕様（Spock用）
  - **workflow/**: ワークフロー系ユースケース（Step Functions）
    - **uc-XXX-*/**: ユースケースごとの設計
      - `workflow-spec.md`: ワークフロー仕様
      - `behavior-spec.md`: 振る舞い仕様（Spock用）
  - **webhook/**: Webhook系ユースケース
    - **uc-XXX-*/**: ユースケースごとの設計
      - `api-spec.md`: API仕様
      - `behavior-spec.md`: 振る舞い仕様（Spock用）
- **adr/**: Architecture Decision Records（設計判断の記録）
- **plans/**: プロジェクト管理・計画関連
  - **roadmap.md**: 開発ロードマップ（Phase別の開発計画）
  - **implementation-notes.md**: 技術的課題と対策
  - **next-actions.md**: 次のアクション（直近のTODO）

### `iac/`
Infrastructure as Code関連のファイルを格納。

#### `iac/pulumi/`
Pulumiによるインフラプロビジョニングコード（TypeScript）。

- **network/**: VPC、Subnet、SecurityGroup等のネットワーク設定
- **database/**: EC2 + Docker PostgreSQLのセットアップ
- **lambda/**: Lambda + API Gatewayの設定
- **tools/maintenance/**: メンテナンスツール

#### `iac/ansible/`
Ansibleによる構成管理コード。

- **provision.yml**: メインのプレイブック
- **group_vars/**: 環境変数設定
- **roles/postgresql/**: PostgreSQL + Dockerのセットアップロール
- **roles/backup-cron/**: バックアップCronジョブの設定ロール

### `backend/` (未作成)
Kotlin + Spring Bootによるバックエンドコード。

### `frontend/` (未作成)
React + TypeScriptによるフロントエンドコード。

## 参考資料

- [システムアーキテクチャ](architecture.md)
- [ADR (Architecture Decision Records)](adr/)
