# データモデル

## エンティティ関係図

```
User ──< Subject
       │        │
       │        └──< Entry
       │               │
       │               ├──< EntryMedia
       │               └──< EntryTag >── Tag
       │
       └──< Tag
```

## テーブル定義

### users（ユーザー）

| カラム名 | 型 | 説明 |
|---|---|---|
| id | UUID (PK) | ユーザーID |
| display_name | VARCHAR(100) | 表示名 |
| email | VARCHAR(255) UNIQUE | メールアドレス（認証用） |
| avatar_url | TEXT | プロフィール画像URL |
| created_at | TIMESTAMPTZ | 作成日時 |
| deleted_at | TIMESTAMPTZ | 退会日時（論理削除） |

### subjects（記録対象）

| カラム名 | 型 | 説明 |
|---|---|---|
| id | UUID (PK) | 記録対象ID |
| owner_id | UUID (FK: users.id) | 管理者ユーザー |
| name | VARCHAR(100) | 対象名（例: 長女、夫、自分） |
| relation | VARCHAR(50) | 続柄・関係 |
| color | VARCHAR(20) | UI上の識別色 |
| created_at | TIMESTAMPTZ | 作成日時 |
| archived_at | TIMESTAMPTZ | アーカイブ日時 |

### entries（日記エントリ）

| カラム名 | 型 | 説明 |
|---|---|---|
| id | UUID (PK) | エントリID |
| owner_id | UUID (FK: users.id) | 記録の所有者 |
| subject_id | UUID (FK: subjects.id) | 記録対象 |
| content | TEXT | 本文 |
| recorded_at | DATE | 記録日（実際の出来事の日） |
| created_at | TIMESTAMPTZ | 投稿日時 |
| updated_at | TIMESTAMPTZ | 最終更新日時 |
| deleted_at | TIMESTAMPTZ | 削除日時（論理削除） |

### entry_media（添付メディア）

| カラム名 | 型 | 説明 |
|---|---|---|
| id | UUID (PK) | メディアID |
| entry_id | UUID (FK: entries.id) | 所属エントリ |
| media_type | ENUM(photo, audio) | メディア種別 |
| storage_key | TEXT | ストレージオブジェクトキー |
| sort_order | INTEGER | 表示順 |
| created_at | TIMESTAMPTZ | アップロード日時 |

### tags（タグ）

| カラム名 | 型 | 説明 |
|---|---|---|
| id | UUID (PK) | タグID |
| owner_id | UUID (FK: users.id) | 所有ユーザー |
| name | VARCHAR(50) | タグ名 |

### entry_tags（エントリ-タグ中間テーブル）

| カラム名 | 型 | 説明 |
|---|---|---|
| entry_id | UUID (FK: entries.id) | - |
| tag_id | UUID (FK: tags.id) | - |

## 設計方針

- **論理削除**: `deleted_at` による論理削除でデータ復元に対応
- **recorded_at と created_at の分離**: 過去の出来事を後から記録できるよう、実際の日付と投稿日時を分ける
- **所有者分離**: v0.1 では `owner_id` を基準にユーザー単位でデータを分離する
- **記録対象の明示**: `subject_id` により、誰についての記録かを明確にする
- **共有拡張を考慮**: 将来的なグループ共有は別テーブルを追加して拡張する
- **UUID**: 外部推測可能な連番IDを避け、UUIDを採用
