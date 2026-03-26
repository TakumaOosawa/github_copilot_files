# Copilot カスタマイズ共通方針

このリポジトリでは、GitHub Copilot の custom agents、custom instructions、agent skills を組み合わせ、工程ごとに責務を分離して運用する。

## 基本原則

- エージェントは単一工程に集中し、前後工程の責務を勝手に取り込まない。
- 作業途中の検討や調整はチャットセッションで行い、永続化するのは正式成果物だけにする。
- 正式成果物、正式入力、handoff は必ず .github/workflow-artifacts/cases/<case-id>/ 配下に保存する。
- workflow-artifacts 直下に sources、outputs、handoffs を直接作らない。必ず cases/<case-id>/ を挟む。
- 詳細設計が存在する工程では、実装判断の主基準を詳細設計とし、基本設計は意図確認用の参考資料として扱う。
- 各工程の完了時には、次工程向けの handoff を必ず 1 つ以上残す。
- 成果物は原則 1 工程 1 正式ファイルとし、.working.md や .final.md に分割しない。
- 不明点、曖昧点、独断禁止事項は本文または handoff に明示する。
- 指示文、テンプレート本文、handoff 本文は原則日本語で記述する。

## 案件ディレクトリ運用

- case-id は小文字英数字とハイフンで命名する。
- 同一案件を継続して改善する場合は同じ case-id を使う。
- 別案件は新しい case-id を作成する。
- 各案件ディレクトリ直下に case-manifest.md を配置し、案件識別情報と現在工程を明示する。

## 品質確認ループ

- コードレビューは終点ではなく、再テストの観点を test-execution に返す品質ゲートとして扱う。
- code-review-to-test-execution.md はレビュー後再テストの正式入力として扱う。
- review-result.md は判定結果、code-review-to-test-execution.md は次工程入力であることを混同しない。

## 作業時の期待動作

- まず対象 case-id を確認し、存在しない場合は新規作成方針を明示する。
- 工程開始時に正式入力を確認し、足りないものがあれば不足を明記する。
- テンプレートの章立てを維持しつつ、入力不足の箇所は空欄で流さず、要確認として明示する。
- 原文忠実性が求められる工程では意訳を避ける。
- Laravel プロジェクトとして、Route、Middleware、Controller、Request、Service、Repository、Model、View の責務分離を意識する。
