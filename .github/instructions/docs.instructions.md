---
description: "Use when creating, updating, or reviewing documentation in docs/. Ensures consistency across planning, design, and development documents."
applyTo: "docs/**"
---

# Documentation — ガイドライン

## ドキュメント間の整合性

変更時は以下の連鎖影響を確認してから編集する。

| 変更対象 | 影響を確認すべきドキュメント |
|---|---|
| `concept.md` | `personas.md` / `requirements.md` / `roadmap.md` |
| `requirements.md` | `roadmap.md` / `api_spec.md` / `data_model.md` |
| `data_model.md` | `api_spec.md`（request/response のフィールド） |
| `tech_stack.md` | `infrastructure.md` / `setup.md` / `runbook.md` |
| `roadmap.md` | `requirements.md`（Must/Should の優先度） |

## 設計原則の一貫性チェック

ドキュメントを編集するとき、以下の方針と整合しているかを確認する。

- **OOUI**: 画面や操作を説明する際は「オブジェクト選択 → 操作」の動線になっているか
- **DDD**: コアドメイン（記録・把握・気遣い）が中心にあるか
- **プライバシー**: 家族情報の漏洩シナリオが含まれていないか
- **フェーズ整合**: 記述内容が現在のフェーズ（v0.1 MVP）に対応しているか

## ADR の作成基準

以下のいずれかに該当するアーキテクチャ決定は `docs/03_development/adr/` に ADR を作成する。

- 技術選定の変更（DB / フレームワーク / インフラ）
- 設計パターンの採用・廃止
- セキュリティ方式の変更
- 将来の移行経路に影響する決定

## 文体・フォーマット

- 日本語で記述する
- `#` はファイルタイトルのみ。本文の見出しは `##` から始める
- 表は Markdown テーブルを使用する
- コードブロックには言語識別子を付ける

## スコープ外の注意

以下の内容は現段階では docs に書かない（将来フェーズの機能）。

- 家族メンバー間のリアルタイム共有・リアクション（v1.x）
- クラウド同期の実装詳細（v1.x）
