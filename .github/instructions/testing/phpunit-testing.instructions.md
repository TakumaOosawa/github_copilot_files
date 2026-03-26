---
name: phpunit-testing
description: PHPUnit テストケースの設計と実施方針を整理するときに使う
applyTo: ".github/workflow-artifacts/cases/*/outputs/testing/*.md"
---
# PHPUnit テストルール

- PHPUnit テストは Laravel プロジェクトの既存構成に合わせて Feature と Unit の責務を分ける。
- HTTP 層や認可、バリデーション、レスポンス確認は Feature テストを優先する。
- 純粋な業務ロジックや分岐条件の検証は Unit テスト候補として整理する。
- 失敗時に原因を特定しやすいよう、入力条件と期待結果を明確にする。
- テストデータの前提、Seeder や Factory 依存、認証状態を明記する。

## ケース整理

- ケース ID
- 対象メソッドまたは機能
- 前提データ
- 実行手順
- 期待結果
- 回帰目的

## 禁止事項

- 1 ケースで複数観点を混在させて失敗原因を不明確にしない。
- テスト観点だけを列挙して、実行条件を省略しない。
- ブラウザ確認が必要な UI 振る舞いを PHPUnit だけで代替したことにしない。
