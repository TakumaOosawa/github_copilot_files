---
name: workflow--common-design-template-guide
description: 基本設計書、詳細設計書、テスト仕様書、レビュー結果、handoff をテンプレートに沿って作成するときに使う。章立て維持と欠落情報の扱い整理に向く。
user-invocable: false
---
# Design Template Guidance

## 使いどころ

- .github/docs/templates 配下のテンプレートに沿って正式成果物を作るとき
- 工程開始時に正式入力の不足有無を確認したいとき
- 章立てを維持しつつ不足情報を要確認として扱いたいとき
- 原文転記すべき章と分析を書く章を分けたいとき

## 基本ルール

- 工程開始時に正式入力を確認し、足りないものがあれば不足として明示する。
- テンプレートの章名は原則維持する。
- 入力不足で埋められない章は空欄にせず、要確認理由を書く。
- 原文忠実性が必要な工程では意訳を避け、本文と補足を分離する。
- Laravel プロジェクトで設計や責務配置を扱う場合は、Route、Middleware、Controller、Request、Service、Repository、Model、View の責務分離を意識する。
- テンプレート本文は原則日本語で記述する。
- handoff は次工程の判断材料になる具体度で書く。

## テンプレート適用手順

1. 対象工程の正式入力を確認する。
2. 対応するテンプレートを開く。
3. 入力から埋められる章を埋める。
4. 推測が必要な箇所は要確認事項へ回す。
5. 参照元、更新日、case-id を文書情報へ記載する。

## 工程別テンプレート

- レビュー工程の review-result.md は `${workspaceFolder}/.github/docs/templates/review-result-template.md` を使用する。
- レビュー工程の review-findings.appendix.md は `${workspaceFolder}/.github/docs/templates/review-findings-appendix-template.md` を使用し、review-result.md に従属する補助明細が必要な場合のみ作成する。

## 出力チェック

- 文書情報があるか
- 参照元が明記されているか
- 章立てが欠落していないか
- 補足と本文が混在していないか
- 次工程が読んで判断できる具体度になっているか
