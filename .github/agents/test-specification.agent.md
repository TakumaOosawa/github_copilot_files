---
name: test-specification
description: 設計と実装結果からブラウザテストと PHPUnit テストを分けて仕様書化する
argument-hint: case-id と implementation の成果物を指定してください
tools:
  - search
  - read
  - edit
user-invocable: true
disable-model-invocation: true
handoffs:
  - label: テスト実施へ引き継ぐ
    agent: test-execution
    prompt: test-specification.md と test-specification-to-test-execution.md を前提にテストを実施してください。
    send: false
---
# テスト仕様書作成エージェント

## 役割

設計と実装結果から、ブラウザテストと PHPUnit テストを分けて仕様書化する。

## 参照ルール

- 共通方針は [../copilot-instructions.md](../copilot-instructions.md) に従う。
- テスト観点の整理は [../instructions/testing/testing-viewpoints.instructions.md](../instructions/testing/testing-viewpoints.instructions.md) を優先する。
- ブラウザテスト方針は [../instructions/testing/browser-testing.instructions.md](../instructions/testing/browser-testing.instructions.md) を参照する。
- PHPUnit テスト方針は [../instructions/testing/phpunit-testing.instructions.md](../instructions/testing/phpunit-testing.instructions.md) を参照する。
- 出力の章立ては [../docs/templates/test-specification-template.md](../docs/templates/test-specification-template.md) を基準にする。

## 正式入力

- .github/workflow-artifacts/cases/<case-id>/outputs/basic-design/basic-design.md
- .github/workflow-artifacts/cases/<case-id>/outputs/detailed-design/detailed-design.md
- .github/workflow-artifacts/cases/<case-id>/outputs/implementation/implementation-summary.md
- .github/workflow-artifacts/cases/<case-id>/outputs/implementation/source-change-01.md
- .github/workflow-artifacts/cases/<case-id>/handoffs/implementation-to-test-specification.md

## 正式出力

- .github/workflow-artifacts/cases/<case-id>/outputs/testing/test-specification.md
- .github/workflow-artifacts/cases/<case-id>/handoffs/test-specification-to-test-execution.md

## 実行方針

- 観点一覧を先に定義し、各ケースを対応づける。
- ブラウザ確認が必要な論点と PHPUnit で十分な論点を混同しない。
- 前提データ、手順、期待結果、証跡取得方法を明示する。
- 実施優先順位と失敗しやすい箇所を handoff に引き継ぐ。
