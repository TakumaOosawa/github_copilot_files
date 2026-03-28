---
name: workflow__common_artifact-location
description: 案件情報一覧ファイル・入力ファイル・成果物ファイル・引継ぎファイルを案件ディレクトリへ配置するときに使う
---
# 案件ファイル配置ルール

- 案件ファイルは必ず `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/` 配下に保存する。
- sources、outputs、handoffsディレクトリを`${workspaceFolder}/.github/workflow-artifacts`直下に直接作成しない。
- case-idが未確定のまま成果物を作成しない。
- 同一案件の継続作業では、既存のcase-idを優先して再利用する。
- 新規案件では、`${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/` 配下にcase-manifest.md、sourcesディレクトリ、outputsディレクトリ、handoffsディレクトリを先に用意する。

## 案件ファイル配置パス

- 案件情報一覧ファイル: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/case-manifest.md`
- 入力ファイル: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/sources/`
  - 基本設計: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/sources/basic-design/`
- 成果物ファイル: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/`
  - 基本設計: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/basic-design/`
  - 詳細設計: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/detailed-design/`
  - 実装: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/implementation/`
  - テスト: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/test/`
  - レビュー: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/review/`
- 引継ぎファイル: `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/`

## 案件ディレクトリ関連の命名ルール

- case-id は小文字英数字とハイフンのみを使う。
- 引継ぎファイル名は、工程のfrom-toが分かる形式にする。
  - 例: detailed-design-to-implementation.md

## 確認事項

- 作業開始前にcase-idを確認する。
  - 既存案件の継続作業では、既存のcase-idを優先して再利用するが、作業前に必ずユーザーに確認する。
  - 既存のcase-idが存在しない場合は、新規案件として、ユーザーに質問してcase-idを決定する。
- 既存案件の再利用時は現在工程を`${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/case-manifest.md`に反映する。
