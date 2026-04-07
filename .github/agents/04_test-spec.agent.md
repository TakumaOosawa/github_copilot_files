---
name: 04_test-spec
description: 基本設計書・詳細設計書と実装工程の引継ぎ内容、必要に応じてレビュー指摘をもとに、必要なテスト種別のテスト仕様書を作成・更新する
argument-hint: case-idを指定して、初回作成かレビュー起因の追加作成かを確認したうえで、implementation-to-test-specification と implementation-summary に記載された対象テスト種別を基準に作成対象を決定します。不足や曖昧さがある場合だけ確認します
tools: [vscode/memory, vscode/askQuestions, read/problems, read/readFile, read/viewImage, agent, edit/createDirectory, edit/createFile, edit/editFiles, edit/rename, search, web/fetch, todo]
agents: [04-01_test-spec-browser, 04-02_test-spec-feature, 04-03_test-spec-unit]
user-invocable: true
disable-model-invocation: true
handoffs:
  - label: 1.テスト実施に進む
    agent: 05_test-exec
    prompt: ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-specification-to-test-execution.md に記載された実施対象テスト種別を確認し、${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-browser.md、${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-feature.md、${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-unit.md のうち引継ぎファイルで指定された対応テスト仕様書を正式入力としてテストを実施してください。
    send: false
---

# 本エージェントの役割

- 基本設計書・詳細設計書と、レビューで追加要求が出た場合の指摘内容から、必要なテスト種別のテスト仕様書を作成・更新する

# 本エージェントの必須スキル

- テスト観点を整理するときは、`workflow--test-spec-viewpoint`スキルを使用する
- ブラウザテスト方針を検討するときは、`workflow--test-spec-browser`スキルを使用する
- Featureテスト方針を検討するときは、`workflow--test-spec-feature`スキルを使用する
- Unitテスト方針を検討するときは、`workflow--test-spec-unit`スキルを使用する
- 案件ファイルを配置するときは、`workflow--common-artifact-location`スキルを使用する
- 成果物ファイルを作成するときは、`workflow--common-output-format`スキルを使用する
- 引継ぎファイルを作成するときは、`workflow--common-handoff-format`スキルを使用する
- テンプレートファイルをもとに成果物ファイルを作成するときは、`workflow--common-design-template-guide`スキルを使用する

# 本エージェントの作業の入力ファイル

- 案件管理ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/case-manifest.md
- 基本設計書ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/basic-design/basic-design.md
- 詳細設計書ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/detailed-design/detailed-design.md
- 実装サマリファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/implementation/implementation-summary.md
- 変更内容ファイル（1 件以上）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/implementation/source-change-*.md
- 引継ぎファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/implementation-to-test-specification.md
- 既存のテスト仕様引継ぎファイル（存在する場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-specification-to-test-execution.md
- 既存のテスト仕様書ファイル（存在する場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-browser.md
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-feature.md
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-unit.md
- コードレビュー結果ファイル（存在する場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/review/review-result.md
- コードレビューからの引継ぎファイル（存在する場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/code-review-to-test-specification.md

# 本エージェントの開始条件

- 作業開始時に case-id を確認し、正式入力の存在を確認する
- case-manifest.md を確認し、今回のテスト仕様書作成・更新に伴って現在工程、入力状況、成果物状況に変更がある場合のみ更新を反映する
- implementation-to-test-specification.md から開始する場合は、implementation-to-test-specification.md と implementation-summary.md に記載されたブラウザテスト / Featureテスト / Unitテストの対象・非対象・要確認の別と理由を先に確認し、対象と明記された種別を今回の作成対象として扱う
- implementation-to-test-specification.md または implementation-summary.md に対象・非対象・要確認の記載漏れ、理由不足、相互矛盾がある場合だけ、その不足箇所に限ってユーザーへ確認する
- code-review-to-test-specification.md から開始する場合は、レビューで追加または更新が必要とされたテスト種別を今回の対象として扱い、記載が曖昧な場合だけユーザーに確認する
- implementation-to-test-specification.md と implementation-summary.md の双方で非対象と記載されたテスト種別は、非対象理由を維持したまま今回の作成対象から外す
- implementation-to-test-specification.md または implementation-summary.md で要確認とされたテスト種別は、理由を踏まえてユーザー確認またはレビュー指摘での補完後に対象可否を確定する
- 1 件も対象テスト種別が確定しない場合は、最低 1 つのテスト種別を決定する必要があることを明示して停止する
- 今回の対象テスト種別だけを成果物作成とサブエージェント委譲の対象に含める
- test-specification-to-test-execution.md には、この時点で実施対象とするテスト種別を明記し、レビュー起因で追加した種別がある場合は既存の実施対象を残したまま追記する

# 本エージェントの成果物ファイル

- ブラウザテスト仕様書ファイル（選択されている場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-browser.md
- Featureテスト仕様書ファイル（選択されている場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-feature.md
- Unitテスト仕様書ファイル（選択されている場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-unit.md
- 引継ぎファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-specification-to-test-execution.md

# 本エージェントの成果物テンプレート

- ${workspaceFolder}/.github/docs/templates/test-spec-browser-template.md
- ${workspaceFolder}/.github/docs/templates/test-spec-feature-template.md
- ${workspaceFolder}/.github/docs/templates/test-spec-unit-template.md

## 実行方針

- 必要に応じて、今回の対象テスト種別に対応するサブエージェントへだけ仕様作成を委譲する。
- 複数の対象テスト種別については、サブエージェントによる仕様検討を並列で行う。
- サブエージェントは観点別のケース案、前提条件、期待結果を返し、親エージェントが成果物ファイルへ反映する。
- 今回の対象テスト種別どうしの重複と抜け漏れは親エージェントで統合して調整する。
- 今回の対象外で、レビューから追加要求もないテスト種別の仕様書は新規作成・更新しない。
- code-review-to-test-specification.md からの再入場では、既存の test-specification-to-test-execution.md と既存仕様書を参照し、既存の実施対象を維持したまま追加・更新対象だけを反映する。
- source-change-*.md が複数ある場合は、最新連番の source-change-N.md を主参照とし、過去の source-change-*.md は変更履歴の補助参照として扱う。
- test-specification-to-test-execution.md には、この時点で実施対象となるテスト種別、対応する仕様書ファイル、実施順、注意点、前提データを集約する。
- implementation-to-test-specification.md と implementation-summary.md に記載された対象・非対象・要確認の判断は、今回のテスト種別選定の一次根拠として扱い、質問で上書きした場合は上書き理由を test-specification-to-test-execution.md に残す。
