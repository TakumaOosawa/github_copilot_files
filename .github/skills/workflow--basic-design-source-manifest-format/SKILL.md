---
name: workflow--basic-design-source-manifest-format
description: 基本設計の source-manifest.md をテンプレートに沿って作成または更新するときに使う
user-invocable: false
---
# source-manifest 記述ルール

- `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/sources/basic-design/source-manifest.md` を作成または更新するときに使う。
- `${workspaceFolder}/.github/docs/templates/source-manifest-template.md` をテンプレートとして使用する。
- テンプレートの章立て、表の列、項目名は維持する。
- 本文は原則日本語で記述する。
- テンプレートには構造だけを置き、補足ルールや読み方の説明はこのスキル側で扱う。
- CSV 一覧には csv ディレクトリ配下のファイルを漏れなく列挙し、意味、シート相当順、備考を記載する。
- 読み方の注意点には例外シート、CSV 化で崩れている可能性がある箇所、先頭からの読み順を記載する。
- 原文だけで確定できない点は独断で補完せず、備考または要確認として残す。
- csv ディレクトリ配下の実データを基本設計 md 化の原文ソースとして扱う。

## 各章の記入指針

- 元資料情報には元の設計書名、作成元、既知の補足情報を記載する。
- CSV 一覧には csv ディレクトリ配下のファイル名を漏れなく列挙し、意味、シート相当順、備考を記載する。
- 読み方の注意点には例外シート、CSV 化で崩れている可能性がある箇所、先頭からの読み順を記載する。
- 補足には本文へ入れにくい前提、制約、要確認事項を記載する。

## 作成・更新手順

1. `${workspaceFolder}/.github/docs/templates/source-manifest-template.md` を開く。
2. 元資料情報に元の設計書名、作成元、既知の補足情報を反映する。
3. csv ディレクトリ配下のファイルを確認し、CSV 一覧へ記載する。
4. 読み順や CSV 化の崩れなど、取り込み時の判断材料を読み方の注意点へ追記する。
5. 未確定情報は断定せず、備考または要確認として残す。