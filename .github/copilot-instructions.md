# nikoroku — Copilot Instructions

家族のリーダーが自分と家族に関するメモを記録・把握するためのメモ日記アプリ。

## 設計原則（常に守ること）

- **OOUI**: 基本動線は「オブジェクトの選択 → 操作」。主オブジェクトは `Subject`・`Entry`・`Tag`
- **DDD**: コアドメイン以外は既存ライブラリを優先。レイヤーは `Domain / Application / Infrastructure`
- **TDD**: 実装より先に失敗するテストを書く。`Red → Green → Refactor`
- **プライバシー優先**: ログに個人情報（日記内容・メールアドレス）を出力しない。データ漏洩ゼロが最優先

## スタック概要

| 対象 | 技術 |
|---|---|
| モバイル | Flutter / Dart、Riverpod、sqflite |
| バックエンド | Go (Gin)、sqlc、DDD 3層構造 |
| DB | PostgreSQL（Aurora Serverless v2） |
| インフラ | AWS ECS Fargate、Terraform |
| IaC | Terraform（S3 + DynamoDB state）|
| CI/CD | GitHub Actions |

## 常に守るルール

- `handler` から `repository` を直接参照しない（`service` 経由）
- セキュリティ変更・スコープ変更・アーキテクチャ決定が必要な場合は必ず人間に確認を求める
- コミットは Conventional Commits 形式（`feat:` / `fix:` / `test:` 等）
- コンソール手動変更禁止。インフラ変更はすべて Terraform 経由

## 現在のフェーズ

v0.1 MVP: ローカル完結アプリ（ネットワーク不要、認証・記録・閲覧が端末内で完結）

## 参照ドキュメント

| ドキュメント | 内容 |
|---|---|
| [docs/01_planning/concept.md](docs/01_planning/concept.md) | コンセプト・ビジョン |
| [docs/01_planning/requirements.md](docs/01_planning/requirements.md) | ユーザーストーリー・機能要件 |
| [docs/01_planning/roadmap.md](docs/01_planning/roadmap.md) | マイルストーン・スコープ |
| [docs/02_design/architecture.md](docs/02_design/architecture.md) | システムアーキテクチャ・設計原則 |
| [docs/02_design/data_model.md](docs/02_design/data_model.md) | エンティティ・テーブル定義 |
| [docs/02_design/api_spec.md](docs/02_design/api_spec.md) | REST API 仕様 |
| [docs/02_design/tech_stack.md](docs/02_design/tech_stack.md) | 技術選定・選定理由 |
| [docs/03_development/coding_conventions.md](docs/03_development/coding_conventions.md) | コーディング規約 |
| [docs/03_development/setup.md](docs/03_development/setup.md) | 開発環境構築 |
| [docs/04_release/checklist.md](docs/04_release/checklist.md) | リリース前チェックリスト |

## スコープ付き Instructions（自動適用）

ファイルを開くと、対応するドメイン別 instruction が自動的に適用されます。

| applyTo pattern | ファイル | 用途 |
|---|---|---|
| `backend/**` | [.github/instructions/backend.instructions.md](.github/instructions/backend.instructions.md) | Go × DDD × TDD ガイド、layer separation、DI pattern |
| `mobile/**` | [.github/instructions/mobile.instructions.md](.github/instructions/mobile.instructions.md) | Flutter × Riverpod × OOUI、state management、sqflite |
| `infra/**` | [.github/instructions/infra.instructions.md](.github/instructions/infra.instructions.md) | Terraform workflow、AWS naming、state backend |
| `docs/**` | [.github/instructions/docs.instructions.md](.github/instructions/docs.instructions.md) | documentation 一貫性、cross-reference、ADR format |

## On-Demand Prompts（手動実行）

Copilot Chat で `/prompt-name` コマンドで起動できる workflow：

| prompt | 用途 |
|---|---|
| `/new-feature` | [.github/prompts/new-feature.prompt.md](.github/prompts/new-feature.prompt.md) | 新機能要件 → domain model → OOUI design → test plan → 実装 checklist |
| `/security-review` | [.github/prompts/security-review.prompt.md](.github/prompts/security-review.prompt.md) | 認証・認可・入力検証・priv policy の security audit |
| `/create-adr` | [.github/prompts/create-adr.prompt.md](.github/prompts/create-adr.prompt.md) | Architecture Decision Record template 生成・リビュー |
| `/release-check` | [.github/prompts/release-check.prompt.md](.github/prompts/release-check.prompt.md) | v?.?.? release 前の final checklist runner |
| `/product-review` | [.github/prompts/product-review.prompt.md](.github/prompts/product-review.prompt.md) | プロダクト論点整理、決定マップ、ギャップ検出、意思決定ログ化 |

## 推奨ワークフロー

### Sprint 開始時
1. `/product-review` で現状決定マップを把握
2. Open 論点の優先度を判定して sprint backlog に追加
3. `/new-feature` で各 story の実装計画を作成

### Feature 実装中
1. `backend/` `mobile/` ファイルを開く → scoped instructions が自動適用
2. TDD cycle: test → implement → `/security-review` で privacy/validation 確認
3. 実装が design 仕様と一致するか domain layer で verify

### Release 前
1. `/release-check` で v?.?.? pre-launch checklist を実行
2. 重要な設計決定は `/create-adr` で log 化
3. Terraform apply は `.github/instructions/infra.instructions.md` の no-console rule を遵守

### 定期的な「決定ログ化」（weekly）
1. `/product-review generate-adr-candidates` で今週の新規決定をスキャン
2. ADR 化の必要性を人間が判定
3. `docs/02_design/adr/` に commit
