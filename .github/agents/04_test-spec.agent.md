---
name: 04_test-spec
description: 基本設計書・詳細設計書と実装工程の引継ぎ内容、必要に応じてレビュー指摘またはテスト実施で判明した不足をもとに、必要なテスト種別のテスト仕様書を作成・更新する
argument-hint: case-idを指定して、指定された対象テスト種別のテスト仕様書を作成・更新します。
tools: [vscode/memory, vscode/askQuestions, read/problems, read/readFile, read/viewImage, agent, edit/createDirectory, edit/createFile, edit/editFiles, edit/rename, search, web/fetch, todo]
agents: [04-01_test-spec-browser, 04-02_test-spec-feature, 04-03_test-spec-unit]
user-invocable: true
disable-model-invocation: true
handoffs:
  - label: 1.詳細設計に戻る
    agent: 02_detailed-design
    prompt: 04_test-specエージェントから作業を引継ぎます。${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/case-manifest.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/basic-design/markdown/*.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/detailed-design/detailed-design.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/implementation/implementation-summary.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/implementation/source-change-*.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-browser.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-feature.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-unit.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-specification-to-detailed-design.md を確認し、テスト仕様作成で判明した設計不足や記述不足を詳細設計へ反映してください。
    send: false
  - label: 2.実装に戻る
    agent: 03_implementation
    prompt: 04_test-specエージェントから作業を引継ぎます。${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/case-manifest.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/detailed-design/detailed-design.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/implementation/implementation-summary.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/implementation/source-change-*.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-browser.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-feature.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-unit.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-specification-to-implementation.md を確認し、テスト仕様作成で判明した実装または実装成果物の不足へ対応してください。
    send: false
  - label: 3.テスト実施に進む
    agent: 05_test-exec
    prompt: 04_test-specエージェントから作業を引継ぎます。${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/case-manifest.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-specification-to-test-execution.md を確認し、${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-browser.md、${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-feature.md、${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-unit.md のうち引継ぎファイルで指定された対応テスト仕様書を正式入力としてテストを実施してください。
    send: false
---

# 本エージェントの役割

- 基本設計書・詳細設計書と実装工程の引継ぎ内容、必要に応じてレビュー指摘をもとに、必要なテスト種別のテスト仕様書を作成・更新する
  - ユーザーと複数回対話しながら、成果物を作成して品質を高める。#tool:vscode/askQuestions を使用して、曖昧な点をユーザーに確認して進める。

# 本エージェントで使用するスキル

- テスト観点を整理するときは、`workflow--test-spec-viewpoint`スキルを使用する
- ブラウザテスト方針を検討するときは、`workflow--test-spec-browser`スキルを使用する
- Featureテスト方針を検討するときは、`workflow--test-spec-feature`スキルを使用する
- Unitテスト方針を検討するときは、`workflow--test-spec-unit`スキルを使用する
- 案件ファイルを配置するときは、`workflow--common-artifact-location`スキルを使用する
- 成果物ファイルを作成するときは、`workflow--common-output-format`スキルを使用する
- 引継ぎファイルを作成するときは、`workflow--common-handoff-format`スキルを使用する
- テンプレートファイルをもとに成果物ファイルを作成するときは、`workflow--common-design-template-guide`スキルを使用する

# 本エージェントの入力

- 案件管理ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/case-manifest.md
- 基本設計書Markdownファイル群
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/basic-design/markdown/*.md
- 詳細設計書ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/detailed-design/detailed-design.md
- 実装要約ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/implementation/implementation-summary.md
- 変更内容ファイル（1 件以上）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/implementation/source-change-*.md
- 前工程からの引継ぎファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/implementation-to-test-specification.md
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/code-review-to-test-specification.md
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-execution-to-test-specification.md
- 既存のテスト仕様書ファイル（存在する場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-browser.md
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-feature.md
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-unit.md
- コードレビュー結果ファイル（存在する場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/review/review-result.md
- テスト結果ファイル（存在する場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-result.md

# 本エージェントの成果物

- ブラウザテスト仕様書ファイル（選択されている場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-browser.md
- Featureテスト仕様書ファイル（選択されている場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-feature.md
- Unitテスト仕様書ファイル（選択されている場合）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-spec-unit.md
- 次工程への引継ぎファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-specification-to-detailed-design.md
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-specification-to-implementation.md
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/test-specification-to-test-execution.md

# 本エージェントの作業フロー

1. 作業開始時に case-id を確認し、03_implementation からの初回着手か 06_review-code からの差し戻し対応か 05_test-exec からの差し戻し対応かを判定したうえで正式入力の存在を確認する
  - 案件管理ファイル、基本設計書Markdownファイル群、詳細設計書ファイル、実装要約ファイル、変更内容ファイル、前工程からの引継ぎファイルなど、今回の作業に必要な正式入力がそろっていることを確認する
  - 05_test-exec からの差し戻し対応として着手する場合は、test-result.md と test-execution-to-test-specification.md も読み込み、テスト実施で判明した仕様不足や誤りを確認する
  - 不足する正式入力がある場合は、ユーザー確認または前工程への差し戻し案内を行い、補完前に成果物作成を進めない
2. 前工程からの引継ぎ内容を確認し、今回のテスト仕様書作成・更新の起点とする判断材料を整理する
  - implementation-to-test-specification.md から開始する場合は、implementation-to-test-specification.md と implementation-summary.md に記載されたブラウザテスト / Featureテスト / Unitテストの対象・非対象・要確認の別と理由を先に確認し、対象と明記された種別を今回の作成対象として扱う
  - implementation-to-test-specification.md または implementation-summary.md に対象・非対象・要確認の記載漏れ、理由不足、相互矛盾がある場合だけ、その不足箇所に限ってユーザーへ確認する
  - implementation-to-test-specification.md と implementation-summary.md の双方で非対象と記載されたテスト種別は、非対象理由を維持したまま今回の作成対象から外す
  - implementation-to-test-specification.md または implementation-summary.md で要確認とされたテスト種別は、理由を踏まえてユーザー確認またはレビュー指摘での補完後に対象可否を確定する
  - code-review-to-test-specification.md から開始する場合は、レビューで追加または更新が必要とされたテスト種別を今回の対象として扱い、記載が曖昧な場合だけユーザーに確認する
  - test-execution-to-test-specification.md から開始する場合は、test-result.md に記録された未実施要因、期待結果不足、前提条件不足、観点不足を確認し、必要な修正対象を今回の対象として扱う
3. 今回の対象テスト種別を確定する
  - 1 件も対象テスト種別が確定しない場合は、最低 1 つのテスト種別を決定する必要があることを明示して停止する
  - 今回の対象テスト種別だけを成果物作成とサブエージェント委譲の対象に含める
4. 変更内容ファイルと既存のテスト仕様書ファイルを確認し、今回の更新範囲を整理する
  - source-change-*.md が複数ある場合は、最新連番の source-change-N.md を主参照とし、過去の source-change-*.md は変更履歴の補助参照として扱う
  - 今回の対象外で、レビューから追加要求もないテスト種別の仕様書は新規作成・更新しない
  - code-review-to-test-specification.md からの再入場では、既存の test-specification-to-test-execution.md と既存仕様書を参照し、既存の実施対象を維持したまま追加・更新対象だけを反映する
  - test-execution-to-test-specification.md からの再入場では、既存仕様書で有効な観点を維持しつつ、不足または誤りがあった観点だけを追記・修正する
5. 必要に応じて、今回の対象テスト種別に対応するサブエージェントへだけ仕様作成を委譲する
  - 複数の対象テスト種別については、サブエージェントによる仕様検討を並列で行う
  - サブエージェントは観点別のケース案、前提条件、期待結果を返し、親エージェントが成果物ファイルへ反映する
6. 今回の対象テスト種別に対応するテスト仕様書ファイルを作成または更新する
  - 今回の対象テスト種別どうしの重複と抜け漏れは親エージェントで統合して調整する
7. 判定に応じて次工程への引継ぎファイルを最新化する
  - 詳細設計の条件、観点、前提が不足しておりテスト仕様へ落とし込めない場合は、test-specification-to-detailed-design.md を作成し、02_detailed-design へ戻す
  - 実装または実装成果物の不足によりテスト対象、前提データ、確認観点を確定できない場合は、test-specification-to-implementation.md を作成し、03_implementation へ戻す
  - テスト実施へ進められる場合は、test-specification-to-test-execution.md を作成し、この時点で実施対象となるテスト種別、対応する仕様書ファイル、実施順、注意点、前提データを集約する
  - レビュー起因またはテスト実施起因で追加した種別がある場合も、既存の実施対象を維持したまま test-specification-to-test-execution.md に追記する
  - test-specification-to-detailed-design.md、test-specification-to-implementation.md、test-specification-to-test-execution.md を同時に正式 handoff として残さない
8. case-manifest.md を確認し、今回のテスト仕様書作成・更新に伴って現在工程、入力状況、成果物状況に変更がある場合のみ更新を反映する
9. 引継ぎ先エージェントの実施に必要な事項を案内する
10. 引継ぎ先エージェントへの引継ぎを案内する

# 本エージェントの禁止事項

- implementation-to-test-specification.md、code-review-to-test-specification.md、test-execution-to-test-specification.md のいずれも確認せずにテスト仕様書作成を開始すること
- 詳細設計または実装の不足が明らかなのに、差し戻し handoff を残さず test-specification-to-test-execution.md だけで進めること
- テスト実施で判明した不足点を既存仕様書へ反映せずに再実施へ戻すこと
