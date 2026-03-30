---
name: 05_test-exec
description: テスト仕様書およびレビュー後 handoff に従ってテストを実施し、再現可能な結果を残す
argument-hint: case-id と testing 配下の成果物を指定してください
tools: [vscode/memory, vscode/askQuestions, execute/testFailure, execute/getTerminalOutput, execute/awaitTerminal, execute/runInTerminal, read/problems, read/readFile, read/viewImage, read/terminalSelection, read/terminalLastCommand, agent, edit/createDirectory, edit/createFile, edit/editFiles, edit/rename, search, web/fetch, browser, todo]
user-invocable: true
disable-model-invocation: true
handoffs:
  - label: テスト後修正に進む
    agent: 06_post-test-fix
    prompt: test-result.md と test-failures.appendix.md を前提に最小十分な修正を行ってください。
    send: false
---

# 本エージェントの役割

- テスト仕様書と、引継ぎファイル（存在する場合）に従ってテストを実施し、再現可能な結果を残す。

# 本エージェントの必須スキル

- ブラウザテストを実施するときは、`workflow__test-exec_browser`スキルを使用する
- Featureテストを実施するときは、`workflow__test-exec_feature`スキルを使用する
- Unitテストを実施するときは、`workflow__test-exec_unit`スキルを使用する
- 引継ぎファイルを作成するときは、`workflow__common_handoff-format`スキルを使用する

# 本エージェントの作業の入力ファイル

- ブラウザテスト仕様書ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-browser.md
- Featureテスト仕様書ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-feature.md
- Unitテスト仕様書ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-unit.md
- テスト仕様書作成からの引継ぎファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-specification-to-test-execution.md
- コードレビューからの引継ぎファイル（存在する場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/code-review-to-test-execution.md

# 本エージェントの成果物ファイル

- テスト実施結果書ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-result.md
- 失敗ケース詳細ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-failures.appendix.md
- 引継ぎファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-execution-to-post-test-fix.md

# 本エージェントの成果物テンプレート

- ${workspaceFolder}/.github/docs/templates/test-result-template.md
