---
name: phpunit-test-execution
description: Laravel の PHPUnit テスト実施方針を Feature と Unit に振り分けるときの共通入口として使う。
user-invocable: false
---
# PHPUnit Test Execution

## 使いどころ

- PHPUnit テストの実施対象が Feature か Unit かを切り分けるとき
- 実行前に HTTP 層確認と業務ロジック確認の責務を整理するとき
- 失敗記録の整理先を Feature / Unit ごとに分けたいとき

## 振り分け方針

1. HTTP リクエストからレスポンスまでの確認が主目的なら workflow__test-exec_feature を使う。
2. クラスやメソッド単位のロジック確認が主目的なら workflow__test-exec_unit を使う。
3. 認証、権限、Route、Middleware、Controller、Request の連携確認は Feature に寄せる。
4. 計算処理、分岐、例外、戻り値の確認は Unit に寄せる。
5. 同じ変更に両観点がある場合は Feature と Unit を分けて記録する。

## 使い分けの目安

- 外部 I/O や認証状態に依存するなら Feature を優先する。
- 外部要因を切り離して再現できるなら Unit を優先する。
- UI 観点が必要な確認は PHPUnit ではなく別のブラウザテスト観点へ分離する。

## 注意事項

- ここには Feature / Unit 共通の振り分け方針だけを置き、実行詳細は各専用スキルに分離する。
- Feature と Unit を 1 つの結果記録で混在させて、失敗原因を曖昧にしない。
- どちらの観点でも、失敗時は再現条件と主要メッセージを省略しない。
