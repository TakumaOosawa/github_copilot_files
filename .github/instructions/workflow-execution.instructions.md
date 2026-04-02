---
description: "Use when starting or progressing any workflow step in this repository. Covers case-id confirmation, required input checks, case-manifest update decisions, template fidelity, and Laravel responsibility separation."
name: "Workflow Execution"
applyTo: "**"
---

# 作業時の期待動作

- まず対象 case-id を確認し、存在しない場合は新規作成方針を明示する。
- 新規案件では _case-template のディレクトリ構成、初期ファイル、テンプレートの意図を確認してから cases/<case-id>/ を作成する。
- 新規案件の case-manifest.md は _case-template/case-manifest.md の初期記入例を起点にする。
- 工程開始時に正式入力を確認し、足りないものがあれば不足を明記する。
- 毎工程の毎操作で case-manifest.md の更新要否を検討し、案件識別情報、現在工程、入力状況、成果物状況に変化があるときだけ反映する。
- source-manifest.md を作成または更新するときは .github/docs/templates/source-manifest-template.md の章立てと記載項目を標準とする。
- テンプレートの章立てを維持しつつ、入力不足の箇所は空欄で流さず、要確認として明示する。
- 原文忠実性が求められる工程では意訳を避ける。
- Laravel プロジェクトとして、Route、Middleware、Controller、Request、Service、Repository、Model、View の責務分離を意識する。
