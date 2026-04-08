---
description: "Use when writing, reviewing, or testing Go backend code. Covers DDD layer structure, TDD workflow, security rules, and Go conventions for nikoroku backend."
applyTo: "backend/**"
---

# Backend (Go) — 開発ガイドライン

## DDD レイヤー構造

```
handler       HTTPリクエスト/レスポンスのみ。ビジネスロジックを含まない
  ↓ (interface経由)
service       ユースケース・ビジネスロジック。ドメインルールをここに集約
  ↓ (interface経由)
repository    データアクセス抽象化。DBの詳細をドメインに漏らさない
  ↓
model         ドメインエンティティと値オブジェクト
```

- `handler` から `repository` を直接呼ばない（service 経由のみ）
- ドメインルールは `model` または `service` に書き、`handler` に書かない

## パッケージ構造

```
backend/
  cmd/             エントリーポイント
  internal/
    handler/       HTTPハンドラー（Ginルーター）
    service/       ユースケース
    repository/    データアクセス（sqlcクエリ利用）
    model/         エンティティ・値オブジェクト
  pkg/             外部公開ユーティリティ
```

## TDD ワークフロー

1. `_test.go` に失敗するテストを先に書く
2. 最小の実装でテストをグリーンにする
3. リファクタリングしてテストを通し続ける
4. バグ修正時は再現テストを追加してから修正する

```go
// 例: service のユニットテストを先に書く
func TestCreateEntry_ShouldFail_WhenContentIsEmpty(t *testing.T) {
    // Arrange
    repo := &mockEntryRepository{}
    svc := service.NewEntryService(repo)
    // Act
    _, err := svc.Create(context.Background(), service.CreateEntryInput{Content: ""})
    // Assert
    assert.ErrorIs(t, err, model.ErrEmptyContent)
}
```

## セキュリティルール

- ユーザー入力は必ずバリデーション・サニタイズしてから使う
- ログに `content`（日記本文）・`email`・シークレット類を出力しない
- 認証チェックは `handler` で行い、`service` でも `owner_id` を必ず検証する
- SQL は sqlc のプレースホルダーバインドのみ使用（生文字列連結禁止）
- シークレットはコードに直書きせず環境変数から取得する

## Go 規約

- フォーマット: `gofmt` / `goimports`（CI で必須）
- リント: `golangci-lint`（CI で必須）
- エラー: `fmt.Errorf("%w", err)` でラップし、呼び出し元でログ
- インターフェースは利用側（service/repository）で定義する
