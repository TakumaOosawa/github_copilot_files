---
name: phpunit-test-execution
description: Laravel の PHPUnit テストを実施し、失敗原因の切り分け、結果整理、再現条件の記録を行うときに使う。Feature と Unit の観点整理にも向く。
user-invocable: false
---
# PHPUnit Test Execution

## 使いどころ

- test-specification.md に基づき PHPUnit を実施するとき
- Feature テストと Unit テストの実施範囲を整理するとき
- 失敗時に test-failures.appendix.md へ再現条件を残すとき

## 実施手順

1. 対象ケースと前提データを確認する。
2. 既存のテスト構成と実行コマンドを確認する。
3. 実行対象を必要に応じて Feature と Unit に分ける。
4. 実行結果から合否、エラーメッセージ、失敗箇所を抽出する。
5. 失敗時は入力条件、認証状態、データ状態、再現性を整理する。
6. 回帰観点がある場合は関連ケースも再確認する。

## 整理項目

- 実行コマンド
- 実行日時
- 合格件数 / 失敗件数
- 失敗ケースの要点
- 原因仮説
- 再テスト要否

## 注意事項

- UI 依存の確認事項を PHPUnit だけで完了扱いにしない。
- 失敗ログは長大でも要約だけにせず、要点となるメッセージを残す。
- DB や認証前提が結果に影響する場合は明記する。
