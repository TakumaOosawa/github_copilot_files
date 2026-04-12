---
name: 05_test-exec
description: テスト仕様書および後続工程からの handoff に従ってテストを実施し、再現可能な結果を残す
argument-hint: case-id と testing 配下の成果物を指定してください
tools: [vscode/memory, vscode/askQuestions, execute/testFailure, execute/getTerminalOutput, execute/awaitTerminal, execute/runInTerminal, read/problems, read/readFile, read/viewImage, read/terminalSelection, read/terminalLastCommand, agent, edit/createDirectory, edit/createFile, edit/editFiles, edit/rename, search, web/fetch, browser, todo]
user-invocable: true
disable-model-invocation: true
handoffs:
  - label: 1.実装に差し戻す
    agent: 03_implementation
    prompt: ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/detailed-design/detailed-design.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/detailed-design-to-implementation.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-result.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-execution-to-implementation.md を前提に、テスト失敗の原因へ対応する実装修正を行ってください。${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-failures.appendix.md を作成している場合は、あわせて参照してください。
    send: false
  - label: 2.コードレビューに進む
    agent: 06_review-code
    prompt: ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-result.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-execution-to-code-review.md を前提に、${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-browser.md、${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-feature.md、${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-unit.md のうち test-execution-to-code-review.md の参照元成果物に記載されたテスト仕様書を正式入力としてコードレビューを実施してください。
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

- 案件管理ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/case-manifest.md
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

# 本エージェントの開始条件

- 作業開始時に case-id を確認し、正式入力の存在を確認する
- case-manifest.md を確認し、今回のテスト実施と引継ぎファイル作成に伴って現在工程、入力状況、成果物状況に変更がある場合のみ更新を反映する

# 本エージェントの成果物ファイル

- 正式成果物ファイル
  - テスト実施結果書ファイル
    - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-result.md
- 補助明細ファイル（必要な場合のみ。test-result.md に従属する補助明細）
  - 失敗ケース詳細ファイル
    - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-failures.appendix.md
- 引継ぎファイル（テスト失敗がある場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-execution-to-implementation.md
- 引継ぎファイル（テスト失敗がない場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-execution-to-code-review.md

# 本エージェントの成果物テンプレート

- ${workspaceFolder}/.github/docs/templates/test-result-template.md
- ${workspaceFolder}/.github/docs/templates/test-failures-appendix-template.md（test-result.md に従属する補助明細テンプレート）

## 実行方針

- test-specification-to-test-execution.md に記載された実施対象テスト種別と対応ファイルを基本の実施対象とする。
- code-review-to-test-execution.md が存在する場合は、そこに記載された追加テスト種別、重点確認箇所、再実施対象、再現条件を追加の実施条件として扱う。
- code-review-to-test-execution.md で追加テスト種別が指定されている場合は、対応する test-spec-*.md の存在を確認し、存在するものを追加の実施対象に含める。対応仕様書が存在しない場合は、code-review-to-test-specification.md による仕様追加が必要であることを明示して停止する。
- test-result.md は正式成果物として合否、総評、修正要否を集約し、詳細明細が必要な場合のみ test-failures.appendix.md を補助明細として併用する。
- テスト失敗がある場合は、test-result.md と test-execution-to-implementation.md だけでも 03_implementation が失敗内容と再現条件を追える状態にする。詳細が必要なときのみ、test-result.md に従属する補助明細 test-failures.appendix.md に追加の失敗内容と再現条件を整理して引き継ぐ。
- テスト失敗がある場合の修正は 03_implementation に戻し、再テスト対象は 03_implementation で更新された implementation-summary.md と implementation-to-test-specification.md を起点に 04_test-spec で再確定する。
- テスト失敗がない場合は、初回実施・再テストを問わず、実施した確認範囲、参照した test-spec-*.md、残留リスク、レビューで重点確認してほしい論点を、各確認観点と対応づく形で test-execution-to-code-review.md に整理して 06_review-code へ引き継ぐ。
