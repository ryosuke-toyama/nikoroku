# ブランチ戦略

## ブランチモデル

GitHub Flow をベースにシンプルに運用する。

```
main ←── feature/xxx
     ←── fix/xxx
     ←── docs/xxx
```

## ブランチ命名規則

| プレフィックス | 用途 | 例 |
|---|---|---|
| `feature/` | 新機能開発 | `feature/timeline-view` |
| `fix/` | バグ修正 | `fix/auth-token-refresh` |
| `docs/` | ドキュメント更新 | `docs/update-readme` |
| `chore/` | 設定・ビルド変更 | `chore/update-dependencies` |

## マージルール

1. `main` ブランチへの直接プッシュは禁止
2. プルリクエスト（PR）を経由してマージする
3. PRのマージ前に以下が必要:
   - CI（テスト・リント）が全て通過
   - レビュアー1名以上の承認
4. マージ方法: **Squash and Merge** を推奨（コミット履歴をきれいに保つ）

## リリース管理

- `main` が常にリリース可能な状態を維持する
- バージョンタグは `v0.1.0` 形式で付与する
- タグ付けは GitHub Releases から行い、`changelog.md` と連動させる
