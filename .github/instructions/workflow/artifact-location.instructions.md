---
name: artifact-location
description: 成果物、入力、handoff を案件ディレクトリへ正しく配置するときに使う
applyTo: ".github/**/*.md"
---
# 成果物配置ルール

- 正式成果物は必ず .github/workflow-artifacts/cases/<case-id>/ 配下に保存する。
- sources、outputs、handoffs を workflow-artifacts 直下に直接作成しない。
- case-id 未確定のまま成果物を確定しない。必要なら case-manifest.md を先に整備する。
- 同一案件の継続作業では既存の case-id を優先して再利用する。
- 新規案件では case-manifest.md、sources、outputs、handoffs の骨格を先に用意する。

## 標準配置

- 案件情報: .github/workflow-artifacts/cases/<case-id>/case-manifest.md
- 外部入力: .github/workflow-artifacts/cases/<case-id>/sources/
- 正式成果物: .github/workflow-artifacts/cases/<case-id>/outputs/
- 引継ぎ: .github/workflow-artifacts/cases/<case-id>/handoffs/

## 命名

- case-id は小文字英数字とハイフンのみを使う。
- output は工程名ディレクトリ配下に 1 成果物 1 ファイルを基本とする。
- handoff ファイル名は from-to が分かる形式にする。例: detailed-design-to-implementation.md

## 確認事項

- 作業開始前に対象 case-id を確認する。
- 既存案件の再利用時は現在工程を case-manifest.md に反映する。
