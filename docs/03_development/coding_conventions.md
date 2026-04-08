# コーディング規約

## 共通

- コミットメッセージは [Conventional Commits](https://www.conventionalcommits.org/) に従う
  - `feat:` 新機能
  - `fix:` バグ修正
  - `docs:` ドキュメントのみの変更
  - `refactor:` リファクタリング
  - `test:` テストの追加・修正
  - `chore:` ビルドやツール設定の変更

## 開発方針（TDD）

- 開発はTDDを原則とし、`Red -> Green -> Refactor` のサイクルで進める
- 新機能は、まず失敗するテストを作成してから実装する
- バグ修正は、再現テストを追加してから修正する
- PRにはテスト追加の有無と理由を明記する
- ドメインロジック変更には必ず単体テストを追加する

## Go（バックエンド）

- フォーマッターは `gofmt` / `goimports` を使用
- リンターは `golangci-lint` を使用（CI必須）
- パッケージ構造:
  ```
  backend/
    cmd/         # エントリーポイント
    internal/
      handler/   # HTTPハンドラー
      service/   # ビジネスロジック
      repository/# データアクセス層
      model/     # エンティティ定義
    pkg/         # 外部公開可能なユーティリティ
  ```
- エラーハンドリング: `errors.New` / `fmt.Errorf("%w", err)` でラップ、呼び出し元でログ
- シークレットはコードに直書きせず、必ず環境変数から取得
- 層構成は DDD を意識し、`handler` から `repository` を直接参照しない

## Dart / Flutter（モバイル）

- フォーマッターは `dart format` を使用
- リンターは `flutter analyze` を使用（CI必須）
- ディレクトリ構造:
  ```
  mobile/lib/
    features/     # 機能ごとのディレクトリ
      timeline/
      entry/
      auth/
    shared/       # 共通ウィジェット・ユーティリティ
    core/         # DI設定、ルーター
  ```
- 状態管理: Riverpod を使用
- ローカルDB: sqflite（オフラインデータ）
- 画面設計は OOUI を前提とし、基本導線を「オブジェクト選択→操作」に統一する

## セキュリティ共通ルール

- ユーザー入力は必ずバリデーション・サニタイズを行う
- 認証・認可チェックはハンドラー側で確実に行い、サービス層でも二重確認する
- ログに個人情報（メールアドレス、日記内容）を出力しない
