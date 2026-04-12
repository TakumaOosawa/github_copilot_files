---
name: workflow--common-implement-summary
description: implementation-summary.md を作成するときの保存先、テンプレート、記載項目、テスト仕様への引継ぎの注意点を統一するときに使う
---

# implementation-summary.md 作成ルール

## このスキルを使う場面

- 03_implementation で implementation-summary.md を新規作成するとき
- 03_implementation で実装内容の変化にあわせて implementation-summary.md を更新するとき

## 保存先

- 保存先は `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/implementation/implementation-summary.md` とする。
- case-id が未確定または曖昧なまま implementation-summary.md を作成しない。

## テンプレート選択

- `.github/docs/templates/implementation-summary-template.md` を使用する。
- テンプレートの見出し構造は維持し、implementation-summary.md の運用ルールはこのスキルを正とする。

## 記載時の注意

- 文書情報には文書名、対象 case-id、作成工程、更新日、参照元を埋める。
- 実装目的には、今回の変更で何を実現したか、なぜ必要だったか、どこまでを対応範囲としたかを分けて記載する。
- 実装概要には、利用者影響、主な変更点、意図的に変更していない事項を区別して記載する。
- 変更対象コンポーネント一覧には、変更対象、変更要点、参照設計を後工程が追跡できる粒度で記載する。
- 設計との整合には、主に参照した詳細設計、整合している点、設計との差分、差分理由を省略せずに記載する。
- テスト仕様への引継ぎでは、ブラウザテスト / Featureテスト / Unitテストの各種別について、対象 / 非対象 / 要確認のいずれかを必ず記載し、理由・根拠を併記する。
- テスト仕様への引継ぎには、重点確認観点、追加・変更した条件分岐、回帰確認が必要な箇所を、次工程がそのまま観点化できる具体さで記載する。
- 懸念事項・要確認には、未解決事項、制約、次工程へ判断を委ねる事項を省略せずに残す。
- テンプレート上の項目に該当事項がない場合も空欄で残さず、不要または該当なしであることを明記する。

## 関連成果物との整合

- implementation-summary.md に記載する変更内容、影響範囲、テスト観点は、同工程で作成または更新した最新の source-change-N.md と追跡できる状態を保つ。
- implementation-summary.md の「テスト仕様への引継ぎ」に記載する各テスト種別の判定と理由は、implementation-to-test-specification.md と矛盾しないように保つ。
- implementation-summary.md は 04_test-spec がテスト種別選定の一次根拠として読む前提で、記載漏れや理由不足がない状態で引き渡す。
