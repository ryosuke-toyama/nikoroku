# 開発環境構築手順

## 開発方式（推奨）

- コード編集: WSL上の VS Code（ホスト）で行う
- CLI操作（依存解決、テスト、マイグレーション、フォーマット）: Docker コンテナ内で行う
- 目的: OS差分やローカル環境差分を最小化し、別PCでも同じ手順で再現できるようにする

このドキュメントでは `docker compose run --rm <service> <command>` を基本操作とする。

## 前提条件

| ツール | 推奨バージョン |
|---|---|
| Docker | 24 以上 |
| Docker Compose | v2 以上 |
| Terraform | 1.7 以上 |
| AWS CLI | 2.x 以上 |
| git | 2.x 以上 |

## リポジトリのクローン

```bash
git clone https://github.com/<your-org>/nikoroku.git
cd nikoroku
```

## コンテナベース開発の前提

- `docker-compose.yml` は、少なくとも以下のサービスを定義する
	- `db`: PostgreSQL
	- `backend-cli`: Go ツールチェーン + マイグレーション実行用
	- `mobile-cli`: Flutter ツールチェーン + テスト実行用
	- `terraform-cli`: Terraform + AWS CLI 実行用
- ソースコードはボリュームマウントし、ホスト編集内容がコンテナ内に即時反映されるようにする
- コンテナ作業用コマンドは、可能な限り `docker compose run --rm ...` に統一する

## バックエンド

```bash
# DB起動
docker compose up -d db

# 依存パッケージのインストール
docker compose run --rm backend-cli go mod download

# 環境変数のセットアップ
cp .env.example .env
# .env を編集して各値を設定

# マイグレーション実行
docker compose run --rm backend-cli go run ./cmd/migrate up

# 開発サーバー起動（ホットリロード: air使用）
docker compose run --rm --service-ports backend-cli air
```

## モバイルアプリ

```bash
# Flutter依存パッケージのインストール
docker compose run --rm mobile-cli flutter pub get

# テスト
docker compose run --rm mobile-cli flutter test

# 解析
docker compose run --rm mobile-cli flutter analyze
```

> 補足: iOSシミュレーター起動や Android エミュレーター起動はホスト依存が強いため、
> 日常のCLI（依存解決・テスト・解析）はコンテナ、実機/エミュレーター実行は必要時のみホストで行う運用を推奨。

## コンテナを使った共通CLIパターン

```bash
# 任意コマンド実行（例: バックエンドのテスト）
docker compose run --rm backend-cli go test ./...

# 任意コマンド実行（例: マイグレーション down）
docker compose run --rm backend-cli go run ./cmd/migrate down 1

# composeを使わない単発実行（必要時）
docker run --rm -v "$PWD":/workspace -w /workspace <image> <command>
```

## 環境変数

`backend/.env.example` を参考に `.env` を作成する。

| 変数名 | 説明 |
|---|---|
| DATABASE_URL | PostgreSQL接続文字列 |
| JWT_SECRET | JWT署名シークレット（32文字以上のランダム文字列） |
| STORAGE_ENDPOINT | R2/S3エンドポイントURL |
| STORAGE_ACCESS_KEY | ストレージアクセスキー |
| STORAGE_SECRET_KEY | ストレージシークレットキー |
| STORAGE_BUCKET | バケット名 |

> **注意**: `.env` ファイルは絶対にGitにコミットしないこと。`.gitignore` に含まれていることを確認する。

## テスト実行

```bash
# バックエンド
docker compose run --rm backend-cli go test ./...

# モバイル
docker compose run --rm mobile-cli flutter test
```

## インフラ（Terraform）

```bash
# AWS認証情報を確認
docker compose run --rm terraform-cli aws sts get-caller-identity

# Terraform 初期化・検証
docker compose run --rm terraform-cli sh -lc '
	cd infra/envs/development &&
	terraform init &&
	terraform fmt -recursive &&
	terraform validate &&
	terraform plan
'
```
