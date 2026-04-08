# API仕様

> 注記: v0.1 はローカル完結を前提とするため、以下はクラウドバックアップや共有拡張を導入する段階での想定APIです。

## 基本仕様

- **プロトコル**: HTTPS (TLS 1.3+)
- **形式**: REST / JSON
- **認証**: Bearer Token (JWT)
- **ベースURL**: `https://api.nikoroku.app/v1`

## 認証

### POST /auth/register
ユーザー登録

**Request Body**
```json
{
  "display_name": "田中さやか",
  "email": "sayaka@example.com",
  "password": "**********"
}
```

**Response 201**
```json
{
  "user": { "id": "uuid", "display_name": "田中さやか" },
  "access_token": "...",
  "refresh_token": "..."
}
```

### POST /auth/login
ログイン

### POST /auth/refresh
アクセストークンの更新

---

## エントリ

### GET /entries
記録一覧の取得

**Query Parameters**
| パラメータ | 型 | 説明 |
|---|---|---|
| from | DATE | 開始日 (YYYY-MM-DD) |
| to | DATE | 終了日 (YYYY-MM-DD) |
| subject_id | UUID | 記録対象でフィルタ |
| tag_id | UUID | タグでフィルタ |
| cursor | string | ページネーションカーソル |
| limit | integer | 取得件数（最大50、デフォルト20） |

**Response 200**
```json
{
  "entries": [
    {
      "id": "uuid",
      "content": "今日は公園で遊んだ。",
      "subject": { "id": "uuid", "name": "長女" },
      "recorded_at": "2026-04-01",
      "media": [{ "id": "uuid", "type": "photo", "url": "https://..." }],
      "tags": [{ "id": "uuid", "name": "子ども" }],
      "created_at": "2026-04-01T21:30:00Z"
    }
  ],
  "next_cursor": "..."
}
```

### POST /entries
エントリの作成

### GET /entries/{id}
エントリの取得

### PATCH /entries/{id}
エントリの更新

### DELETE /entries/{id}
エントリの削除（論理削除）

---

## 記録対象

### GET /subjects
記録対象一覧の取得

### POST /subjects
記録対象の作成

**Request Body**
```json
{
  "name": "長女",
  "relation": "child"
}
```

### PATCH /subjects/{id}
記録対象の更新

### DELETE /subjects/{id}
記録対象の削除

---

## メディア

### POST /entries/{id}/media
メディアのアップロード（multipart/form-data）

### DELETE /entries/{entry_id}/media/{media_id}
メディアの削除

---

## エクスポート

### POST /exports
全データのエクスポートをリクエスト

**Request Body**
```json
{ "format": "json" }
```

**Response 202**: エクスポートジョブをキューに登録し、完了時に通知

---

## 共有拡張（将来機能） 

### POST /groups
グループの作成

### POST /groups/join
招待コードでグループに参加

### GET /groups/{id}/members
メンバー一覧の取得

---

## エラーレスポンス形式

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "認証が必要です"
  }
}
```

| HTTPステータス | コード | 説明 |
|---|---|---|
| 400 | VALIDATION_ERROR | リクエストパラメータ不正 |
| 401 | UNAUTHORIZED | 認証エラー |
| 403 | FORBIDDEN | 権限エラー |
| 404 | NOT_FOUND | リソースが存在しない |
| 429 | RATE_LIMITED | レートリミット超過 |
| 500 | INTERNAL_ERROR | サーバー内部エラー |
