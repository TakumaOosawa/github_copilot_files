---
name: 00_case-init
description: 新規案件ディレクトリを作成して初期化する
argument-hint: 新規案件のcase-idを指定してください。
tools: [vscode/memory, vscode/askQuestions, read/problems, read/readFile, read/viewImage, agent, edit/createDirectory, edit/createFile, edit/editFiles, edit/rename, search, web/fetch, todo]
user-invocable: true
disable-model-invocation: true
handoffs:
  - label: 1.基本設計取り込みに進む
    agent: 01_basic-design-import
    prompt: ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/case-manifest.mdと ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/sources/basic-design/source-manifest.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/case-init-to-basic-design-import.md を確認し、${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/sources/basic-design/csv/ 配下の CSV 形式の基本設計書を Markdown へ取り込んでください。
    send: false
---

# 本エージェントの役割

- 新規案件ディレクトリを作成して初期化する

# 本エージェントで使用するスキル

- 案件ディレクトリにファイルを配置するときは、`workflow--common-artifact-location`スキルを使用する
- case-manifest.md を作成または更新するときは、`workflow--common-case-manifest-format`スキルを使用する
- source-manifest.md を作成または更新するときは、`workflow--basic-design-source-manifest-format`スキルを使用する
- 引継ぎファイルを作成するときは、`workflow--common-handoff-format`スキルを使用する

# 本エージェントの入力

- 案件ID（case-001, 認証画面の追加, etc.）
- 案件ディレクトリ構成テンプレート
  - ${workspaceFolder}/.github/docs/templates/_case-template/
- 案件管理ファイルテンプレート
  - ${workspaceFolder}/.github/docs/templates/case-manifest-template.md
- 基本設計書一覧ファイルテンプレート
  - ${workspaceFolder}/.github/docs/templates/source-manifest-template.md
- 引継ぎファイルテンプレート
  - ${workspaceFolder}/.github/docs/templates/handoff-template.md

# 本エージェントの成果物

- 案件管理ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/case-manifest.md
- 基本設計書一覧ファイル
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/sources/basic-design/source-manifest.md
- 引継ぎファイル（案件初期化 → 基本設計書取り込み）
  - ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/case-init-to-basic-design-import.md

# 本エージェントの開始条件

- 作業開始時に、対象が既存 case-id を再利用すべき継続案件ではなく、新規案件起票であることを確認する
- ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/ が既に存在する場合は上書きせず、既存案件か case-id 衝突かを要確認として停止する

# 本エージェントの作業要件

- .github/docs/templates/_case-template のディレクトリ構成を確認し、cases/<case-id>/ 配下へ sources/basic-design/csv/、outputs/basic-design/、outputs/detailed-design/、outputs/implementation/、outputs/testing/、outputs/review/、handoffs/ を用意する
- case-manifest.md は `workflow--common-case-manifest-format` スキルと .github/docs/templates/case-manifest-template.md を使用し、案件名、case-id、開始日、現在工程、ステータス、入力状況、成果物状況を初期状態へ更新する
- source-manifest.md は `workflow--basic-design-source-manifest-format` スキルと .github/docs/templates/source-manifest-template.md を使用し、既知情報を反映し、未確定項目は要確認として明示する
- 基本設計 CSV が未配置の場合でも csv ディレクトリは作成し、case-manifest.md と handoff へ未配置であることを明記する
- case-init-to-basic-design-import.md には、作成したディレクトリ、CSV 配置状況、01_basic-design-import が着手前に確認すべき未解決事項を記載する

- 新規案件ディレクトリを`${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/`に作成する。`${workspaceFolder}/.github/docs/templates/_case-template`に準拠して初期化する
- 以下を準備して、01_basic-design-importエージェントが開始できる状態にする
  - ファイルの新規作成と初期化
    - `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/case-manifest.md`
    - `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/sources/basic-design/source-manifest.md`
  - ディレクトリの新規作成
    - `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/sources/basic-design/csv/`
    - `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/`
    - `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/basic-design/`
    - `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/detailed-design/`
    - `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/implementation/`
    - `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/`
    - `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/review/`
    - `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/`

# 本エージェントの禁止事項

- .github/docs/templates/_case-template 配下を実案件として編集すること
- 既存案件の case ディレクトリを新規案件として上書きすること
- case-id や案件情報が未確定のまま case-manifest.md を確定させること
- CSV 未配置を隠したまま 01_basic-design-import へ引き継ぐこと
