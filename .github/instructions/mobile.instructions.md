---
description: "Use when writing, reviewing, or testing Flutter/Dart mobile code. Covers OOUI screen design, Riverpod state management, sqflite local DB, TDD, and privacy rules."
applyTo: "mobile/**"
---

# Mobile (Flutter/Dart) — 開発ガイドライン

## OOUI 画面設計

基本導線: **「オブジェクトの選択 → 操作」**

1. まず Subject（記録対象）を選ぶ or 一覧として提示する
2. オブジェクトを選択したあとに、取れる操作（作成・編集・削除・検索）を提示する
3. 操作ボタンは選択状態に応じて活性/非活性を変化させる
4. 一覧画面は Subject / Entry / Tag の主オブジェクトを中心に設計する

```
NG: ホーム画面に「記録する」ボタンをいきなり置く
OK: Subject 一覧 → 対象を選ぶ → 「この人の記録を追加」ボタンが現れる
```

## ディレクトリ構造

```
mobile/lib/
  features/
    subject/     記録対象（OOUI主オブジェクト①）
    entry/       記録（OOUI主オブジェクト②）
    auth/        認証・アプリロック
  shared/        共通ウィジェット・ユーティリティ
  core/          DI設定（Riverpod）、ルーター
```

## 状態管理 (Riverpod)

- UI（Widget）にビジネスロジックを書かない。Provider / Notifier に分離する
- `AsyncValue` を正しく扱う（loading / error / data の 3 状態すべて）
- `StateNotifier` は単一責任を保つ（1 Notifier = 1 ユースケース）

## ローカル DB (sqflite)

- v0.1 はすべての永続化を sqflite で行う（オフライン完結）
- スキーマ変更はマイグレーション関数で管理し、直接 DROP TABLE しない
- エンティティと DB の変換は Repository クラスに隔離する

## TDD ワークフロー

1. Widget テスト: 画面描画・タップ操作を先にテストで定義する
2. Unit テスト: Notifier / Provider のロジックを単体でテストする
3. `flutter test` が CI で必須パスすること

## プライバシー

- `debugPrint` / `print` に日記内容・メールアドレスを出力しない
- `flutter_secure_storage` でトークン・認証情報を保存する
- ローカル DB の暗号化が将来必要になるケースを意識した設計を心がける

## フォーマット・リント

- `dart format`（コミット前に実行）
- `flutter analyze`（CI で必須）
