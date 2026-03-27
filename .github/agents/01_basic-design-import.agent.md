---
name: 01_basic-design-import
description: CSV形式の基本設計書をテンプレートに従ってMarkdownファイルに変換する
argument-hint: case-idを指定して、CSV形式の基本設計書を読み込みます
tools: [vscode/memory, vscode/askQuestions, read/problems, read/readFile, read/viewImage, agent, edit/createDirectory, edit/createFile, edit/editFiles, edit/rename, search, web/fetch, todo]
user-invocable: true
disable-model-invocation: true
handoffs:
  - label: 詳細設計作成に進む
    agent: detailed-design
    prompt: ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/basic-design/basic-design.md と ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/basic-design-to-detailed-design.md をもとに、詳細設計を作成してください。
    send: false
---

# 役割

- CSV形式の基本設計書をテンプレートに従ってMarkdownファイルに変換する

# 必須スキル

- 成果物配置は`workflow__common_artifact-location`スキルを使用する
- 原文忠実性は`workflow__basic-design_fidelity`スキルを使用する
- 出力の章立ては [../docs/templates/basic-design-md-template.md](../docs/templates/basic-design-md-template.md) を基準にする。
- handoff の書式は [../instructions/workflow/handoff-format.instructions.md](../instructions/workflow/handoff-format.instructions.md) を参照する。

## 正式入力

- ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/case-manifest.md
- ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/sources/basic-design/source-manifest.md
- ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/sources/basic-design/csv/*.csv

## 正式出力

- ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/basic-design/basic-design.md
- ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/handoffs/basic-design-to-detailed-design.md

## 実行方針

- まず case-id と正式入力の有無を確認する。
- CSV は原文ソースとして扱い、意訳や実装補完を避ける。
- 本文は原則転記し、補足や懸念は専用章へ分離する。
- 読替えが必要な箇所は根拠を明記して扱う。
- 曖昧点、読み取り不能箇所、独断禁止事項は handoff に残す。

## 禁止事項

- 詳細設計レベルの責務分解を先回りしない。
- Laravel 実装方式を推測して本文へ混ぜない。
- workflow-artifacts 直下に成果物を作らない。
