---
description: "Use when working in this repository on any workflow task. Covers core workflow principles, artifact handling, handoff expectations, and Japanese authoring rules."
name: "Workflow Core"
applyTo: "**"
---

# ワークフロー基本原則

- エージェントは単一工程に集中し、前後工程の責務を勝手に取り込まない。
- 作業途中の検討や調整はチャットセッションで行い、永続化するのは正式成果物だけにする。
- 正式成果物、正式入力、handoff は必ず .github/workflow-artifacts/cases/<case-id>/ 配下に保存する。
- workflow-artifacts 直下に sources、outputs、handoffs を直接作らない。必ず cases/<case-id>/ を挟む。
- 詳細設計が存在する工程では、実装判断の主基準を詳細設計とし、基本設計は意図確認用の参考資料として扱う。
- 各工程の完了時には、次工程向けの handoff を必ず 1 つ以上残す。
- 成果物は原則 1 工程 1 正式ファイルとし、.working.md や .final.md に分割しない。
- 不明点、曖昧点、独断禁止事項は本文または handoff に明示する。
- 指示文、テンプレート本文、handoff 本文は原則日本語で記述する。
