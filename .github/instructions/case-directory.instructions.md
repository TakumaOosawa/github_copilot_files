---
description: "Use when creating or updating workflow artifact directories for a case. Covers case-id rules, case directory layout, case-manifest, and basic-design source placement."
name: "Case Directory"
applyTo: "**"
---

# 案件ディレクトリ運用

- case-id は小文字英数字とハイフンで命名する。
- 同一案件を継続して改善する場合は同じ case-id を使う。
- 別案件は新しい case-id を作成する。
- 新規案件では cases/<case-id>/ を .github/workflow-artifacts/cases/_case-template/ にならって作成する。
- case-manifest.md の初期記入は .github/workflow-artifacts/cases/_case-template/case-manifest.md を記入例として踏襲する。
- 各案件ディレクトリ直下に case-manifest.md を配置し、案件識別情報、現在工程、入力状況、成果物状況を明示する。
- 基本設計書 CSV は cases/<case-id>/sources/basic-design/csv/ に配置する。
- cases/<case-id>/sources/basic-design/source-manifest.md を用意し、標準テンプレートは .github/docs/templates/source-manifest-template.md を参照する。
