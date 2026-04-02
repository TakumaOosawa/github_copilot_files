# Copilot カスタマイズ共通方針

このリポジトリでは、GitHub Copilot の custom agents、custom instructions、agent skills を組み合わせ、工程ごとに責務を分離して運用する。

詳細ルールは .github/instructions/ 配下の .instructions.md ファイルに分割して管理する。

## 分割先

- .github/instructions/workflow-core.instructions.md
- .github/instructions/case-directory.instructions.md
- .github/instructions/quality-loop.instructions.md
- .github/instructions/workflow-execution.instructions.md

このファイルは、ワークスペース全体の入口として残す。
