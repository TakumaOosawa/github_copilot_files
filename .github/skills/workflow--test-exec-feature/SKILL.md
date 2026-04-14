---
name: workflow--test-exec-feature
description: Laravel の Featureテストを実施し、HTTP 層の失敗原因を切り分け、結果整理と再現条件を記録するときに使う。
user-invocable: false
---
# Feature Test Execution

## 使いどころ

- test-spec-feature.md に基づき Featureテストを実施するとき
- Route、Middleware、Controller、Request、認可を含む HTTP 層の確認をするとき
- 失敗時に test-failures.appendix.md へ認証状態や前提データを残すとき

## 実施手順

1. 対象ケース、対象エンドポイント、認証状態、権限前提を確認する。
2. Seeder、Factory、DB 状態などの前提データを確認する。
3. 対象の Featureテスト構成と実行コマンドを確認する。
4. リクエスト、レスポンス、バリデーション、認可結果を中心に実行結果を確認する。
5. 失敗時は入力条件、認証状態、データ状態、失敗箇所、再現性を整理する。
6. 回帰観点がある場合は同一エンドポイント周辺の関連ケースも再確認する。

## 整理項目

- 実行コマンド
- 実行日時
- 合格件数 / 失敗件数
- 失敗ケースの要点
- 認証・権限前提
- 前提データ
- 原因仮説
- 再テスト要否

## 成果物テンプレート

- test-result.md は .github/docs/templates/test-result-template.md をテンプレートとして作成または更新する。
- test-failures.appendix.md は .github/docs/templates/test-failures-appendix-template.md をテンプレートとして作成または更新する。

## 注意事項

- UI 依存の見た目確認を Featureテストだけで完了扱いにしない。
- 失敗ログは長大でも要約だけにせず、要点となるレスポンスや例外メッセージを残す。
- DB、認証、権限前提が結果に影響する場合は必ず明記する。
- Unit テスト向きの純粋ロジック確認まで Featureテストに抱え込まない。
