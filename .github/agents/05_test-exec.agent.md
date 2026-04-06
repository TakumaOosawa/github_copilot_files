---
name: 05_test-exec
description: テスト仕様書および後続工程からの handoff に従ってテストを実施し、再現可能な結果を残す
argument-hint: case-id と testing 配下の成果物を指定してください
tools: [vscode/memory, vscode/askQuestions, execute/testFailure, execute/getTerminalOutput, execute/awaitTerminal, execute/runInTerminal, read/problems, read/readFile, read/viewImage, read/terminalSelection, read/terminalLastCommand, agent, edit/createDirectory, edit/createFile, edit/editFiles, edit/rename, search, web/fetch, browser, todo]
user-invocable: true
disable-model-invocation: true
handoffs:
  - label: 1.テスト後修正に進む
    agent: 06_post-test-fix
    prompt: ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-result.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-failures.appendix.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-execution-to-post-test-fix.md を前提に最小十分な修正を行ってください。
    send: false
  - label: 2.コードレビューに進む
    agent: 07_review-code
    prompt: ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-result.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-execution-to-code-review.md を前提にコードレビューを実施してください。
    send: false
---

# 本エージェントの役割

- テスト仕様書と、引継ぎファイル（存在する場合）に従ってテストを実施し、再現可能な結果を残す。

# 本エージェントの必須スキル

- ブラウザテストを実施するときは、`workflow--test-exec-browser`スキルを使用する
- Featureテストを実施するときは、`workflow--test-exec-feature`スキルを使用する
- Unitテストを実施するときは、`workflow--test-exec-unit`スキルを使用する
- 引継ぎファイルを作成するときは、`workflow--common-handoff-format`スキルを使用する
- テンプレートファイルをもとに成果物ファイルを作成するときは、`workflow--common-design-template-guide`スキルを使用する

# 本エージェントの作業の入力ファイル

- ブラウザテスト仕様書ファイル（選択されている場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-browser.md
- Featureテスト仕様書ファイル（選択されている場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-feature.md
- Unitテスト仕様書ファイル（選択されている場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-unit.md
- テスト仕様書作成からの引継ぎファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-specification-to-test-execution.md
- コードレビューからの引継ぎファイル（存在する場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/code-review-to-test-execution.md
- テスト後修正からの引継ぎファイル（存在する場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/post-test-fix-to-test-execution.md

# 本エージェントの成果物ファイル

- テスト実施結果書ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-result.md
- 失敗ケース詳細ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-failures.appendix.md
- 引継ぎファイル（テスト失敗がある場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-execution-to-post-test-fix.md
- 引継ぎファイル（テスト失敗がない場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-execution-to-code-review.md

# 本エージェントの成果物テンプレート

- ${workspaceFolder}/.github/docs/templates/test-result-template.md

## 実行方針

- test-specification-to-test-execution.md に記載された実施対象テスト種別と対応ファイルを基本の実施対象とする。
- code-review-to-test-execution.md または post-test-fix-to-test-execution.md が存在する場合は、そこに記載された追加テスト種別、重点確認箇所、再実施対象、再現条件を追加の実施条件として扱う。
- code-review-to-test-execution.md で追加テスト種別が指定されている場合は、対応する test-spec-*.md の存在を確認し、存在するものを追加の実施対象に含める。対応仕様書が存在しない場合は、code-review-to-test-specification.md による仕様追加が必要であることを明示して停止する。
- 06_post-test-fix の後続では、post-test-fix-to-test-execution.md を正式入力として必ず再テストを完了してからレビューへ進む。
- テスト失敗がある場合は、失敗内容と再現条件を test-failures.appendix.md と test-execution-to-post-test-fix.md に整理して 06_post-test-fix へ引き継ぐ。
- テスト失敗がない場合は、初回実施・再テストを問わず、実施した確認範囲、残留リスク、レビューで重点確認してほしい論点を test-execution-to-code-review.md に整理して 07_review-code へ引き継ぐ。
