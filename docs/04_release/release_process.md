# リリース手順

## バージョニング

[セマンティックバージョニング](https://semver.org/lang/ja/) に従う: `MAJOR.MINOR.PATCH`

- **MAJOR**: 後方互換性のないAPI変更
- **MINOR**: 後方互換性のある新機能追加
- **PATCH**: バグ修正

## リリースフロー

### 1. バージョンの決定
- 変更内容を確認し、次のバージョン番号を決める

### 2. changelog.md の更新
- `[Unreleased]` セクションを新バージョンセクションに変換
- リリース日を記載

### 3. バージョンタグの付与
```bash
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```

### 4. GitHub Releases の作成
- タグを選択し、changelog の内容をリリースノートに貼付

### 5. バックエンドのデプロイ
```bash
# 先に Terraform でインフラ差分を適用
terraform -chdir=infra/envs/production fmt -recursive
terraform -chdir=infra/envs/production validate
terraform -chdir=infra/envs/production plan
terraform -chdir=infra/envs/production apply

# GitHub Actions による自動デプロイ（タグ push で発動）
# 手動デプロイの場合:
aws ecr get-login-password | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
docker build -t nikoroku-api .
docker push <account-id>.dkr.ecr.<region>.amazonaws.com/nikoroku-api:latest
aws ecs update-service --cluster nikoroku --service nikoroku-api --force-new-deployment
```

### 6. モバイルアプリのビルド・提出

#### iOS
```bash
cd mobile
flutter build ipa
# Xcodeで Archive → Distribute App → App Store Connect
```

#### Android
```bash
cd mobile
flutter build appbundle
# Google Play Console にアップロード
```

### 7. App Store / Google Play の審査待ち
- iOS: 通常1〜3日
- Android: 通常数時間〜1日

### 8. リリース確認
- 本番環境での動作確認（チェックリスト参照）
- 異常があればロールバック手順を実行

## ロールバック手順

### バックエンド（ECS Fargate）
```bash
# まずは Terraform の直前適用状態を確認
terraform -chdir=infra/envs/production plan

# 旧タスク定義リビジョンに戻す
aws ecs update-service \
  --cluster nikoroku \
  --service nikoroku-api \
  --task-definition nikoroku-api:<previous-revision> \
  --force-new-deployment
```

### データベースマイグレーション
```bash
go run ./cmd/migrate down 1
```
