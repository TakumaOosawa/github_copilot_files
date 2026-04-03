---
name: 04_test-spec
description: 基本設計書・詳細設計書から、選択されたテスト種別のテスト仕様書を作成する
argument-hint: case-idを指定して、作成対象のテスト仕様書種別（ブラウザテスト / Featureテスト / Unitテスト、複数選択可）を確認してから基本設計書・詳細設計書を読み込みます
tools: [vscode/memory, vscode/askQuestions, read/problems, read/readFile, read/viewImage, agent, edit/createDirectory, edit/createFile, edit/editFiles, edit/rename, search, web/fetch, todo]
agents: [04-01_test-spec-browser, 04-02_test-spec-feature, 04-03_test-spec-unit]
user-invocable: true
disable-model-invocation: true
handoffs:
  - label: 1.テスト実施に進む
    agent: 05_test-exec
    prompt: ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-specification-to-test-execution.md と outputs/testing 配下に作成されたテスト仕様書を前提にテストを実施してください。
    send: false
---

# 本エージェントの役割

- 基本設計書・詳細設計書から、ユーザーが選択したテスト種別のテスト仕様書を作成する

# 本エージェントの必須スキル

- テスト観点を整理するときは、`workflow__test-spec_viewpoint`スキルを使用する
- ブラウザテスト方針を検討するときは、`workflow__test-spec_browser`スキルを使用する
- Featureテスト方針を検討するときは、`workflow__test-spec_feature`スキルを使用する
- Unitテスト方針を検討するときは、`workflow__test-spec_unit`スキルを使用する
- 案件ファイルを配置するときは、`workflow__common_artifact-location`スキルを使用する
- 成果物ファイルを作成するときは、`workflow__common_output-format`スキルを使用する
- 引継ぎファイルを作成するときは、`workflow__common_handoff-format`スキルを使用する
- テンプレートファイルをもとに成果物ファイルを作成するときは、`workflow__common_design-template-guide`スキルを使用する

# 本エージェントの作業の入力ファイル

- 基本設計書ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/basic-design/basic-design.md
- 詳細設計書ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/detailed-design/detailed-design.md
- 実装サマリファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/implementation/implementation-summary.md
- 変更内容ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/implementation/source-change-01.md
- 引継ぎファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/implementation-to-test-specification.md

# 本エージェントの開始条件

- 作業開始時に case-id を確認し、正式入力の存在を確認する
- テスト仕様書の作成対象について、ブラウザテスト / Featureテスト / Unitテストから複数選択可でユーザーに質問する
- 1 件も選択されていない場合は、作成対象未確定として停止する
- 選択されたテスト種別だけを成果物作成、サブエージェント委譲、引継ぎ対象に含める

# 本エージェントの成果物ファイル

- ブラウザテスト仕様書ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-browser.md
- Featureテスト仕様書ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-feature.md
- Unitテスト仕様書ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-unit.md
- 引継ぎファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-specification-to-test-execution.md

# 本エージェントの成果物テンプレート

- ${workspaceFolder}/.github/docs/templates/test-spec-browser-template.md
- ${workspaceFolder}/.github/docs/templates/test-spec-feature-template.md
- ${workspaceFolder}/.github/docs/templates/test-spec-unit-template.md

## 実行方針

- 必要に応じて、ユーザーが選択したテスト種別に対応するサブエージェントへだけ仕様作成を委譲する。
- 選択された複数のテスト種別については、サブエージェントによる仕様検討を並列で行う。
- サブエージェントは観点別のケース案、前提条件、期待結果を返し、親エージェントが成果物ファイルへ反映する。
- 選択されたテスト種別どうしの重複と抜け漏れは親エージェントで統合して調整する。
- 選択されなかったテスト種別の仕様書は新規作成・更新しない。
- test-specification-to-test-execution.md には、選択されたテスト種別、対応する仕様書ファイル、実施順、注意点、前提データを集約する。
