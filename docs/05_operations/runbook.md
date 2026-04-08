# 障害対応ランブック

## 初動フロー

```
障害検知（Sentry / 監視アラート / ユーザー報告）
  └─ 影響範囲の確認
        ├─ 軽微（一部機能に影響）→ 修正PRを作成してデプロイ
        └─ 重大（サービス全体に影響）→ 以下の手順に従う
```

## サービス全体障害

### 1. 状況確認
```bash
# ECS サービスのステータス確認
aws ecs describe-services \
  --cluster nikoroku --services nikoroku-api

# ログ確認（CloudWatch Logs）
aws logs tail /ecs/nikoroku-api --follow

# DB接続確認（RDS Proxy 経由）
psql "$DATABASE_URL" -c 'SELECT 1'
```

### 2. 直近のデプロイを確認
```bash
aws ecs describe-task-definition --task-definition nikoroku-api
```

### 3. 直前バージョンへのロールバック
```bash
# 旧タスク定義リビジョンを指定して強制デプロイ
aws ecs update-service \
  --cluster nikoroku \
  --service nikoroku-api \
  --task-definition nikoroku-api:<previous-revision> \
  --force-new-deployment
```

---

## DBの応答遅延・接続エラー

1. DB接続数を確認
   ```sql
   SELECT count(*) FROM pg_stat_activity;
   ```
2. 長時間クエリを確認・強制終了
   ```sql
   SELECT pid, now() - pg_stat_activity.query_start AS duration, query
   FROM pg_stat_activity
   WHERE state = 'active'
   ORDER BY duration DESC;

   SELECT pg_terminate_backend(<pid>);
   ```
3. 必要に応じてAPIサーバーを再起動
   ```bash
   aws ecs update-service \
     --cluster nikoroku \
     --service nikoroku-api \
     --force-new-deployment
   ```

---

## ストレージ（R2）障害

1. Cloudflare Status (https://www.cloudflarestatus.com/) を確認
2. 障害が Cloudflare 側の場合は復旧を待つ
3. 写真アップロード機能は一時的に無効化（フィーチャーフラグで制御）

---

## セキュリティインシデント（不正アクセス疑い）

1. 対象ユーザーのセッションを即時無効化（JWTの `jti` ブラックリストに追加、または全ユーザーのリフレッシュトークンをローテーション）
2. アクセスログを保全（削除しない）
3. 影響範囲を特定
4. 必要に応じてユーザーに通知・パスワードリセットを促す
5. 根本原因を調査し、修正・再発防止策を実施

---

## エスカレーション

現状: 個人開発のため、すべてのインシデント対応は開発者が行う。
連絡先: プロジェクトオーナーのメールアドレス（別途 `.env` / シークレットに保管）
