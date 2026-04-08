---
description: "Use when writing, reviewing, or planning Terraform configuration or AWS infrastructure changes. Covers Terraform workflow, AWS resource naming, and security rules for nikoroku infrastructure."
applyTo: "infra/**"
---

# Infrastructure (Terraform / AWS) — 開発ガイドライン

## Terraform 運用ルール

- AWSリソースの作成・変更・削除はすべて Terraform 経由で行う
- **コンソール手動変更は禁止**（terraform state との乖離が発生するため）
- 変更フロー: `fmt` → `validate` → `plan` → **人間が承認** → `apply`
- 破壊的変更（`-/+`）は staging に適用して確認してから production に進む

```bash
# 標準変更フロー
docker compose run --rm terraform-cli sh -lc '
  cd infra/envs/production &&
  terraform fmt -recursive &&
  terraform validate &&
  terraform plan -out=tfplan
'
# plan 内容を人間が確認してから apply
docker compose run --rm terraform-cli sh -lc '
  cd infra/envs/production && terraform apply tfplan
'
```

## ディレクトリ構造

```
infra/
  modules/
    ecs/          ECS Fargate タスク・サービス定義
    rds/          Aurora Serverless v2
    networking/   VPC・サブネット・ALB・セキュリティグループ
    monitoring/   CloudWatch ダッシュボード・アラーム
  envs/
    development/  ローカル開発向け（最小構成）
    staging/      結合テスト環境
    production/   本番環境
```

## ステート管理

```hcl
terraform {
  backend "s3" {
    bucket         = "nikoroku-tfstate"
    key            = "envs/production/terraform.tfstate"
    region         = "ap-northeast-1"
    dynamodb_table = "nikoroku-tfstate-lock"
    encrypt        = true
  }
}
```

## AWS リソース命名規則

- パターン: `nikoroku-{env}-{resource}`
- 例: `nikoroku-prod-ecs-api` / `nikoroku-prod-rds` / `nikoroku-prod-alb`

## セキュリティ

- IAM ポリシーは最小権限原則（必要なアクションのみ許可）
- セキュリティグループはポートを明示的に制限する（0.0.0.0/0 を使わない）
- RDS はプライベートサブネットのみ配置し、`publicly_accessible = false`
- シークレット類は AWS Secrets Manager または SSM Parameter Store で管理する
