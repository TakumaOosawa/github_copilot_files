# Copilot カスタマイズ共通方針

このリポジトリでは、GitHub Copilot の custom agents と agent skills を組み合わせ、工程ごとに責務を分離して運用する。

詳細ルールは .github/skills/ 配下の workflow--* スキルに分割して管理する。各エージェントは必要なスキルを明示して使用する。

## 主な共通スキル

- .github/skills/workflow--common-artifact-location/SKILL.md
- .github/skills/workflow--common-output-format/SKILL.md
- .github/skills/workflow--common-implement-summary/SKILL.md
- .github/skills/workflow--common-implement-source-change/SKILL.md
- .github/skills/workflow--common-design-template-guide/SKILL.md
- .github/skills/workflow--common-handoff-format/SKILL.md
- .github/skills/workflow--implementation-authority/SKILL.md
- .github/skills/workflow--review-code-scope/SKILL.md
- .github/skills/workflow--common-case-manifest-format/SKILL.md
- .github/skills/workflow--basic-design-source-manifest-format/SKILL.md

このファイルは、ワークスペース全体の入口として残す。
