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
