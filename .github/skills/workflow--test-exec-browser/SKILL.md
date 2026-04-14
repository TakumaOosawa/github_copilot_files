---
name: workflow--test-exec-browser
description: 統合ブラウザを使った画面テスト、操作確認、証跡整理を行うときに使う。ブラウザテスト仕様の実施、画面遷移確認、入力検証、スクリーンショット有無の整理に向く。
user-invocable: false
---
# Browser Test Execution

## 使いどころ

- 統合ブラウザで Web アプリの動作確認を実施するとき
- フォーム入力、遷移、ボタン操作、表示崩れを確認するとき
- test-result.md や test-failures.appendix.md に残す証跡を整理するとき

## 実施手順

1. 対象テストケースの前提条件、入力値、期待結果を確認する。
2. 必要なブラウザツールが有効であることを確認する。
3. エージェントが開いたページ、またはユーザーが共有したページを明確化する。
4. readPage を起点に画面構造を把握する。
5. click、type、hover、drag、dialog 操作を必要最小限で組み合わせる。
6. 期待結果と実結果を差分で記録する。
7. 失敗時は再現手順、入力値、スクリーンショット有無を明記する。

## 重点観点

- 正常操作で期待画面になるか
- 異常入力で適切な検証メッセージが出るか
- リンクやボタンが意図した遷移先を開くか
- 主要画面要素が欠落していないか
- レスポンシブ崩れや明らかな UI 崩れがないか

## 記録ルール

- test-result.md は .github/docs/templates/test-result-template.md をテンプレートとして作成または更新する。
- test-failures.appendix.md は .github/docs/templates/test-failures-appendix-template.md をテンプレートとして作成または更新する。
- ケースごとに操作手順を簡潔に残す。
- 失敗時はどの操作で何が起きたかを時系列で残す。
- スクリーンショットを取得しなかった場合でも、その理由を必要に応じて残す。
