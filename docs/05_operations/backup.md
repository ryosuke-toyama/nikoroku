# バックアップ・リストア手順

## バックアップ方針

| 対象 | 頻度 | 保持期間 | 方法 |
|---|---|---|---|
| PostgreSQL | 毎日 0:00 UTC | 30日間 | RDS 自動バックアップ（スナップショット） |
| Cloudflare R2（写真・音声） | 随時（バージョニング） | 無期限 | R2 オブジェクトバージョニング有効化 |

## PostgreSQL バックアップ

### 手動バックアップの取得
```bash
# RDS スナップショットを手動作成
aws rds create-db-snapshot \
  --db-instance-identifier nikoroku-db \
  --db-snapshot-identifier nikoroku-manual-$(date +%Y%m%d)

# ローカルへのダンプ（踏み台経由 or SSM Session Manager）
pg_dump "$DATABASE_URL" > backup_$(date +%Y%m%d).sql
```

### バックアップ一覧の確認
```bash
aws rds describe-db-snapshots \
  --db-instance-identifier nikoroku-db \
  --query 'DBSnapshots[*].{Id:DBSnapshotIdentifier,Time:SnapshotCreateTime}'
```

## リストア手順

### PostgreSQL のリストア
```bash
# RDS スナップショットから新規インスタンスを起動
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier nikoroku-db-restore \
  --db-snapshot-identifier <snapshot-id>

# SQLダンプからのリストア
psql "$DATABASE_URL" < backup_YYYYMMDD.sql
```

### R2 オブジェクトのリストア
```bash
# 特定バージョンのオブジェクトを取得
aws s3api get-object \
  --bucket nikoroku-media \
  --key <object-key> \
  --version-id <version-id> \
  output_file.jpg \
  --endpoint-url https://<account-id>.r2.cloudflarestorage.com
```

## ユーザーデータエクスポート

ユーザーが自分のデータをエクスポートする機能は API 経由で提供。  
詳細は [API仕様](../02_design/api_spec.md) の `POST /groups/{id}/export` を参照。

## バックアップの定期確認

月次で以下を確認する:
- [ ] RDS 自動バックアップ（スナップショット）が正常に完了しているか
- [ ] バックアップからの試験リストアを実施して復元可能かを確認
- [ ] R2 オブジェクトバージョニングが有効になっているか
