---
name: workflow--common-implement-source-change
description: source-change-N.md を作成するときの採番、テンプレート選択、履歴保持、関連成果物との整合を統一するときに使う
---

# source-change-N.md 作成ルール

## このスキルを使う場面

- 03_implementation で source-change-N.md を新規作成するとき
- 03_implementation でテスト実施差し戻し対応として source-change-N.md を新規作成するとき
- 既存の source-change-*.md を確認して今回の N を決めるとき

## 保存先

- 保存先は `${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/outputs/implementation/source-change-N.md` とする。
- case-id が未確定または曖昧なまま source-change-N.md を作成しない。

## 採番ルール

- source-change-N.md は実装変更履歴の正式成果物として扱う。
- 既存の source-change-*.md は履歴として保持し、上書きしない。
- N は 2 桁の連番で採番する。
- 既存の source-change-*.md が存在しない場合は 01 を使用する。
- 既存の source-change-*.md が存在する場合は、既存の最大連番 + 1 を未使用の次連番として採番する。
- 03_implementation が test-execution-to-implementation.md を正式入力として着手する場合も、N を 02 固定にせず、既存の source-change-*.md を確認して今回の N を決める。
- 03_implementation が test-execution-fix-analysis-N.md を同一ループで作成する場合は、source-change-N.md と同じ N を使用する。

## テンプレート選択

- 03_implementation が詳細設計またはレビュー差し戻しを正式入力として着手する場合は `.github/docs/templates/source-change-01-template.md` を使用する。
- 03_implementation が test-execution-to-implementation.md を正式入力として着手する場合は `.github/docs/templates/source-change-02-template.md` を使用する。
- テンプレートの見出し構造は維持し、source-change-N.md の運用ルールはこのスキルを正とする。

## 記載時の注意

- 文書情報には文書名、対象 case-id、作成工程、更新日、参照元を埋める。
- 変更ファイル一覧には、変更対象、変更区分、変更内容、関連設計または関連指摘を後工程が追える粒度で記載する。
- 変更詳細には、変更前と変更後、理由、影響範囲、利用者影響または再発防止観点を省略せずに記載する。
- 見送り事項、残留リスク、要確認事項がある場合は省略せず、次工程が判断に使える形で残す。
- テストで確認してほしい点や再テストで見るべき観点は、後工程がそのまま確認項目へ落とせる具体さで記載する。
- テンプレート上の項目に該当事項がない場合も空欄で残さず、不要または該当なしであることを明記する。

## 関連成果物との整合

- 03_implementation では、今回追加した最新の source-change-N.md を同工程で更新する関連成果物から参照できる状態を保つ。
- 03_implementation が test-execution-to-implementation.md を正式入力として着手した場合は、implementation-summary.md と implementation-to-test-specification.md から、今回追加した最新の source-change-N.md と必要に応じて対応する test-execution-fix-analysis-N.md を追える状態を保つ。
