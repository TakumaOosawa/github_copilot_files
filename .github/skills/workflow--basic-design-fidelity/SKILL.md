---
name: workflow--basic-design-fidelity
description: CSV形式の基本設計書を原文忠実にMarkdownファイルに変換するときに使用する
---

# 基本設計書Markdown変換ルール

- 基本設計書Markdown変換の主目的は構造化であり、意味の変更ではない。
- 以下のパスのCSVファイル群の基本設計書原文を第1ソースとし、意訳や実装寄りの補完を避ける。
  - `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/sources/basic-design/csv/*.csv`
- 項目名、見出し、制約、備考は原文の語彙をできるだけ維持する。
- シート相当の区切りや文書順は `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/case-manifest.md` に従う。
- 判読が難しい箇所や、CSV由来の表示崩れは「原文補足が必要な箇所」に分離して記載する。
- 解釈が分かれる記述は本文に断定反映せず、「要確認事項」に記載する。

## 補足の書き方

- 補足は本文に混ぜず、補足章へ分離する。
- 推定や言い換えをした場合は、必ず根拠となる原文位置を明記する。

## 禁止事項

- 詳細設計レベルの責務分解を先回りして書かない。
- 実装方式やLaravel構成を推測追加しない。
- 原文にないバリデーションや例外処理を勝手に定義しない。
