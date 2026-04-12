---
name: workflow--common-artifact-location
description: 案件情報一覧ファイル・入力ファイル・成果物ファイル・引継ぎファイルを案件ディレクトリへ配置するときに使う
---
# 案件ファイル配置ルール

- 案件ファイルは必ず `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/` 配下に保存する。
- workflow-artifacts 直下に sources、outputs、handoffs を直接作成しない。必ず cases/<case-id>/ を挟む。
  - case-idはユーザーとのやり取りを通して確定させている値を使用する。曖昧な場合は、ユーザーに確認してから作業する。
  - case-idが誤った値のまま作業を進めると、案件ファイルが見つからなくなったり、案件ファイルが散在して管理が煩雑になるリスクがあるため、case-idが未確定・曖昧のまま案件ファイルを作成しない。
  - case-idが未確定のまま成果物を作成しない。

## 案件ファイル一覧（ここで定義している名称を正式名称として使用する）

- 案件管理ファイル: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/case-manifest.md`
- 入力ファイルディレクトリ: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/sources/`
  - 基本設計書ディレクトリ: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/sources/basic-design/`
  - 基本設計書CSV群ディレクトリ: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/sources/basic-design/csv/`
  - 基本設計書一覧ファイル: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/sources/basic-design/source-manifest.md`
- 成果物ファイルディレクトリ: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/`
  - 基本設計成果物ディレクトリ: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/basic-design/`
  - 詳細設計成果物ディレクトリ: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/detailed-design/`
  - 実装成果物ディレクトリ: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/implementation/`
  - テスト成果物ディレクトリ: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/testing/`
  - レビュー成果物ディレクトリ: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/review/`
- 引継ぎファイルディレクトリ: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/`

## 案件ディレクトリ関連の命名ルール

- case-idは、ディレクトリ名として使用できる文字はすべて指定できる。
  - ただし、ユーザーが指定した値ではディレクトリが作成できない場合は、ユーザーに確認してから作業する。
- 引継ぎファイル名は、工程のfrom-toが分かる形式にする。
  - 例: detailed-design-to-implementation.md

## 確認事項

- 作業開始前にcase-idを確認する。
  - 既存案件の継続作業では、既存のcase-idを優先して再利用するが、あいまいな場合は作業前にユーザーに確認する。
  - 既存のcase-idが存在しない場合は、新規案件として、ユーザーに質問してcase-idを決定する。
- 工程開始時に正式入力の所在を確認し、足りないものがあれば不足入力として明示する。
- 新規案件では`${workspaceFolder}/.github/docs/templates/_case-template/`のディレクトリ構成と、`${workspaceFolder}/.github/docs/templates/case-manifest-template.md`、`${workspaceFolder}/.github/docs/templates/source-manifest-template.md`を確認し、必要な初期ファイルとディレクトリをもれなく用意する。
- 各案件の`${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/case-manifest.md`は`${workspaceFolder}/.github/docs/templates/case-manifest-template.md`をテンプレートとして使用し、`workflow--common-case-manifest-format`スキルに従って作成・更新する。
- 各案件の`${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/sources/basic-design/source-manifest.md`は`${workspaceFolder}/.github/docs/templates/source-manifest-template.md`をテンプレートとして使用し、`workflow--basic-design-source-manifest-format`スキルに従って作成・更新する。
- 毎工程の毎操作で`${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/case-manifest.md`の、案件識別情報、現在工程、入力状況、成果物状況を最新化する。
