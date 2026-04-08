# インフラ構成

## 本番環境構成

| コンポーネント | サービス | 備考 |
|---|---|---|
| APIサーバー | AWS ECS Fargate | コンテナ管理不要、タスク単位でスケール |
| データベース | AWS RDS (Aurora Serverless v2 for PostgreSQL) | managed PostgreSQL、停止時ゼロスケール対応 |
| ストレージ | Cloudflare R2 | 写真・音声ファイル（転送料無料） |
| CDN | Cloudflare | 静的アセット配信 |
| 監視 | AWS CloudWatch + Sentry | メトリクス・ログ・エラー追跡 |
| IaC | Terraform | AWSリソース定義をコード化して再現性を担保 |
| CI/CD | GitHub Actions | テスト・ECS への自動デプロイ |

## 環境区分

| 環境 | 用途 | デプロイトリガー |
|---|---|---|
| development | ローカル開発 | 手動 |
| staging | 結合テスト・QA | PRマージ → `develop` ブランチ |
| production | 本番サービス | タグ付け (`v*.*.*`) |

## ネットワーク設計

```
インターネット
  │
  └─ Cloudflare (CDN / DDoS保護)
        │
        └─ AWS ALB (TLS終端)
              │
              └─ ECS Fargate タスク（APIサーバーコンテナ）
                    └─ RDS Aurora Serverless（VPCプライベートサブネット）
```

## スケーリング方針

- **現在（個人・家族利用）**: ECS Fargate タスク最小構成（0.25vCPU / 512MB）、Aurora Serverless 最低 ACU で待機
- **将来（ユーザー増加時）**: ECS タスク数の増加、Aurora ACU の上限緩和で対応
- DB接続は RDS Proxy 経由でコネクションプーリングを行う

## 永続化先選定メモ（RDS vs DynamoDB）

- 現在のデータモデルは `entries`、`tags`、`entry_tags` の関係が強く、RDB前提で整合性を取りやすい
- DynamoDB は低トラフィック時のコスト優位がある一方、クエリ設計の事前固定が必要で開発コストが増える
- 学習コストと要件適合性を優先し、現時点では RDS を採用する
- 四半期ごとに、実トラフィックと月額コストを見て DynamoDB 移行余地を再評価する

## IaC運用方針

- AWSインフラは Terraform で管理し、コンソール手動変更は原則禁止とする
- Terraform のステートはリモートバックエンド（S3 + DynamoDBロック）で管理する
- 変更フローは `terraform fmt` → `terraform validate` → `terraform plan` → 承認後 `terraform apply`
- 破壊的変更は staging で検証後に production へ適用する

## コスト見積もり（月次概算）

| サービス | 費用 |
|---|---|
| ECS Fargate（最小構成） | 約 $5〜10/月 |
| RDS Aurora Serverless v2（停止活用） | 約 $0〜10/月 |
| ALB | 約 $2〜5/月 |
| CloudWatch（基本メトリクス） | 無料枠内 |
| Cloudflare R2 | 無料〜（転送無料、保存 $0.015/GB/月） |
| GitHub Actions | 無料枠内 |
| **合計** | **約 $10〜25/月** |
