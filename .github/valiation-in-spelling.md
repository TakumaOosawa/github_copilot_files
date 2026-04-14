# .github 配下 GitHub Copilot 設定ファイル 表記ゆれ調査

## 調査概要

- 対象
  - .github/copilot-instructions.md
  - .github/agents 配下
  - .github/skills 配下
  - .github/docs/templates 配下
- 実施方法
  - runSubagent を 10 本使って観点ごとに並列調査
  - 候補を grep_search と read_file で再検証
- 判定基準
  - 同じ対象を指しているのに別表記になっているものを採用
  - テンプレート名と生成物名のように別オブジェクトであるものは原則除外

## 調査に使った 10 観点

1. 上流工程エージェント群の呼称揺れ
2. テスト仕様エージェント群の呼称揺れ
3. テスト実施・レビューエージェント群の呼称揺れ
4. common 系 skill の用語揺れ
5. 設計系 skill の用語揺れ
6. テスト系 skill の用語揺れ
7. templates 配下の文書名・欄名の揺れ
8. frontmatter 名と実ファイル参照の整合
9. 日本語用語の横断スイープ
10. typo とスペース差分の横断スイープ

## 結論

- 確度高の表記ゆれは 8 件
- 参考扱いの揺れ候補は 3 件
- frontmatter の name と実ファイル名、skill 名と参照名の対応は概ね整合していた

## 確度高の指摘

### 1. implementation-summary.md の呼称が 実装要約 と 実装サマリ で混在している

- 揺れ
  - 実装要約ファイル
  - 実装サマリファイル
- 主な確認箇所
  - [agents/03_implementation.agent.md](agents/03_implementation.agent.md#L29)
  - [agents/03_implementation.agent.md](agents/03_implementation.agent.md#L44)
  - [agents/03_implementation.agent.md](agents/03_implementation.agent.md#L67)
  - [agents/04_test-spec.agent.md](agents/04_test-spec.agent.md#L48)
  - [agents/04_test-spec.agent.md](agents/04_test-spec.agent.md#L81)
  - [docs/templates/implementation-summary-template.md](docs/templates/implementation-summary-template.md#L1)
- 同一対象と判断した理由
  - いずれも outputs/implementation/implementation-summary.md を指している
- 推奨正規形
  - 実装要約ファイル
- 判断理由
  - template 名が 実装要約テンプレート であり、skill 名も common-implement-summary に対応しているため

### 2. 差し戻し と 差戻し が同一ファイル内でも混在している

- 揺れ
  - 差し戻し
  - 差戻し
- 主な確認箇所
  - [agents/03_implementation.agent.md](agents/03_implementation.agent.md#L79)
  - [agents/03_implementation.agent.md](agents/03_implementation.agent.md#L82)
  - [agents/03_implementation.agent.md](agents/03_implementation.agent.md#L83)
  - [agents/03_implementation.agent.md](agents/03_implementation.agent.md#L84)
  - [agents/03_implementation.agent.md](agents/03_implementation.agent.md#L87)
  - [agents/03_implementation.agent.md](agents/03_implementation.agent.md#L92)
  - [skills/workflow--common-implement-source-change/SKILL.md](skills/workflow--common-implement-source-change/SKILL.md#L11)
  - [docs/templates/test-result-template.md](docs/templates/test-result-template.md#L6)
- 同一対象と判断した理由
  - いずれも前工程へ戻す手戻り対応を指す運用用語として使われている
- 推奨正規形
  - 差し戻し
- 判断理由
  - リポジトリ全体では 差し戻し のほうが優勢で、同一ファイル内でもこちらが併存している

### 3. Featureテスト と Feature テスト が混在している

- 揺れ
  - Featureテスト
  - Feature テスト
- 主な確認箇所
  - [agents/04_test-spec.agent.md](agents/04_test-spec.agent.md#L33)
  - [agents/05_test-exec.agent.md](agents/05_test-exec.agent.md#L31)
  - [docs/templates/test-spec-feature-template.md](docs/templates/test-spec-feature-template.md#L1)
  - [docs/templates/implementation-summary-template.md](docs/templates/implementation-summary-template.md#L37)
  - [skills/workflow--test-spec-feature/SKILL.md](skills/workflow--test-spec-feature/SKILL.md#L5)
  - [skills/workflow--test-spec-viewpoint/SKILL.md](skills/workflow--test-spec-viewpoint/SKILL.md#L7)
  - [skills/workflow--test-spec-viewpoint/SKILL.md](skills/workflow--test-spec-viewpoint/SKILL.md#L29)
  - [agents/04-01_test-spec-browser.agent.md](agents/04-01_test-spec-browser.agent.md#L12)
  - [skills/workflow--test-exec-feature/SKILL.md](skills/workflow--test-exec-feature/SKILL.md#L3)
- 同一対象と判断した理由
  - いずれも Feature test という同一のテスト種別を指している
- 推奨正規形
  - Featureテスト
- 判断理由
  - テンプレート名、親 agent、skill 名、implementation-summary の判定表で無スペース表記が多数派

### 4. Unitテスト と Unit テスト が混在している

- 揺れ
  - Unitテスト
  - Unit テスト
- 主な確認箇所
  - [agents/04_test-spec.agent.md](agents/04_test-spec.agent.md#L34)
  - [agents/05_test-exec.agent.md](agents/05_test-exec.agent.md#L32)
  - [docs/templates/test-spec-unit-template.md](docs/templates/test-spec-unit-template.md#L1)
  - [docs/templates/implementation-summary-template.md](docs/templates/implementation-summary-template.md#L38)
  - [skills/workflow--test-spec-unit/SKILL.md](skills/workflow--test-spec-unit/SKILL.md#L5)
  - [skills/workflow--test-spec-viewpoint/SKILL.md](skills/workflow--test-spec-viewpoint/SKILL.md#L7)
  - [skills/workflow--test-spec-viewpoint/SKILL.md](skills/workflow--test-spec-viewpoint/SKILL.md#L38)
  - [agents/04-01_test-spec-browser.agent.md](agents/04-01_test-spec-browser.agent.md#L12)
  - [skills/workflow--test-exec-unit/SKILL.md](skills/workflow--test-exec-unit/SKILL.md#L3)
- 同一対象と判断した理由
  - いずれも Unit test という同一のテスト種別を指している
- 推奨正規形
  - Unitテスト
- 判断理由
  - Featureテスト と同様に、テンプレート名と主要 agent/skill では無スペース表記が多数派

### 5. test-result.md の呼称が テスト結果 と テスト実施結果書 で揺れている

- 揺れ
  - テスト結果ファイル
  - テスト実施結果ファイル
  - テスト実施結果書テンプレート
  - テスト実施結果書ファイル
- 主な確認箇所
  - [agents/03_implementation.agent.md](agents/03_implementation.agent.md#L56)
  - [agents/04_test-spec.agent.md](agents/04_test-spec.agent.md#L62)
  - [agents/05_test-exec.agent.md](agents/05_test-exec.agent.md#L53)
  - [agents/05_test-exec.agent.md](agents/05_test-exec.agent.md#L60)
  - [agents/05_test-exec.agent.md](agents/05_test-exec.agent.md#L87)
  - [agents/06_review-code.agent.md](agents/06_review-code.agent.md#L4)
  - [docs/templates/test-result-template.md](docs/templates/test-result-template.md#L1)
- 同一対象と判断した理由
  - いずれも outputs/testing/test-result.md を指している
- 推奨正規形
  - テスト結果ファイル
  - テンプレートを指す場合は テスト結果テンプレート
- 判断理由
  - 実ファイル名が test-result.md であり、template 側も テスト結果テンプレート を採用しているため

### 6. case-id 欄のラベルが case-id と 対象 case-id で分かれている

- 揺れ
  - case-id
  - 対象 case-id
- 主な確認箇所
  - [docs/templates/case-manifest-template.md](docs/templates/case-manifest-template.md#L3)
  - [docs/templates/basic-design-template.md](docs/templates/basic-design-template.md#L5)
  - [docs/templates/detailed-design-template.md](docs/templates/detailed-design-template.md#L5)
  - [docs/templates/implementation-summary-template.md](docs/templates/implementation-summary-template.md#L5)
  - [docs/templates/handoff-template.md](docs/templates/handoff-template.md#L5)
  - [docs/templates/test-result-template.md](docs/templates/test-result-template.md#L5)
- 同一対象と判断した理由
  - どちらも案件識別子の入力欄であり、意味は同一
- 推奨正規形
  - 対象 case-id
- 判断理由
  - case-manifest-template.md 以外のテンプレートでは 対象 case-id が採用されている

### 7. agent 見出しが 本エージェントの成果物 と 本エージェントの成果物ファイル で揺れている

- 揺れ
  - 本エージェントの成果物
  - 本エージェントの成果物ファイル
- 主な確認箇所
  - [agents/00_case-init.agent.md](agents/00_case-init.agent.md#L43)
  - [agents/01_basic-design-import.agent.md](agents/01_basic-design-import.agent.md#L46)
  - [agents/02_detailed-design.agent.md](agents/02_detailed-design.agent.md#L53)
  - [agents/03_implementation.agent.md](agents/03_implementation.agent.md#L65)
  - [agents/04_test-spec.agent.md](agents/04_test-spec.agent.md#L65)
  - [agents/05_test-exec.agent.md](agents/05_test-exec.agent.md#L58)
  - [agents/06_review-code.agent.md](agents/06_review-code.agent.md#L66)
- 同一対象と判断した理由
  - いずれも agent が出力する成果物の一覧セクション
- 推奨正規形
  - 本エージェントの成果物
- 判断理由
  - 7 件中 5 件でこちらが使われており、ファイルの語を足さなくても意味が通る

### 8. agent 見出しの Markdown 記法が #本 と # 本 で揺れている

- 揺れ
  - #本エージェントで使用するスキル
  - # 本エージェントで使用するスキル
- 主な確認箇所
  - [agents/02_detailed-design.agent.md](agents/02_detailed-design.agent.md#L24)
  - [agents/00_case-init.agent.md](agents/00_case-init.agent.md#L20)
  - [agents/01_basic-design-import.agent.md](agents/01_basic-design-import.agent.md#L24)
  - [agents/03_implementation.agent.md](agents/03_implementation.agent.md#L24)
  - [agents/04_test-spec.agent.md](agents/04_test-spec.agent.md#L29)
  - [agents/05_test-exec.agent.md](agents/05_test-exec.agent.md#L28)
  - [agents/06_review-code.agent.md](agents/06_review-code.agent.md#L33)
- 同一対象と判断した理由
  - 同じ見出しを Markdown で記述しているだけで、対象セクションは完全に同一
- 推奨正規形
  - # 本エージェントで使用するスキル
- 判断理由
  - 7 件中 6 件で採用されており、Markdown としてもこちらが標準的

## 参考扱いの揺れ候補

### 9. ブラウザテスト と 画面テスト

- 主な確認箇所
  - [skills/workflow--test-exec-browser/SKILL.md](skills/workflow--test-exec-browser/SKILL.md#L3)
  - [skills/workflow--test-spec-browser/SKILL.md](skills/workflow--test-spec-browser/SKILL.md#L3)
  - [agents/04-01_test-spec-browser.agent.md](agents/04-01_test-spec-browser.agent.md#L3)
- コメント
  - browser 系 skill の description 内で 画面テスト と ブラウザテスト が同居している
  - ただし 画面テスト は説明語で、厳密にはテスト種別名そのものではない可能性があるため参考扱いに留める
- 推奨
  - テスト種別名としては ブラウザテスト に寄せる

### 10. 規約 と コーディング規約

- 主な確認箇所
  - [agents/06_review-code.agent.md](agents/06_review-code.agent.md#L3)
  - [agents/06_review-code.agent.md](agents/06_review-code.agent.md#L95)
  - [skills/workflow--review-code-scope/SKILL.md](skills/workflow--review-code-scope/SKILL.md#L7)
  - [agents/06-01_review-coding-rules.agent.md](agents/06-01_review-coding-rules.agent.md#L11)
- コメント
  - レビュー観点の一覧では 規約 と省略されるが、subagent 名と skill 本文では コーディング規約 を使っている
  - 同義として読めるため参考扱いに留める
- 推奨
  - レビュー観点のラベルは コーディング規約 に統一すると説明の粒度が揃う

### 11. 詳細設計書ファイル と 詳細設計ファイル

- 主な確認箇所
  - [agents/03_implementation.agent.md](agents/03_implementation.agent.md#L40)
  - [agents/04_test-spec.agent.md](agents/04_test-spec.agent.md#L46)
  - [agents/06_review-code.agent.md](agents/06_review-code.agent.md#L47)
  - [skills/workflow--detailed-design-structure/SKILL.md](skills/workflow--detailed-design-structure/SKILL.md#L6)
- コメント
  - 06_review-code だけが 詳細設計ファイル で、他は 詳細設計書ファイル を採用している
  - 参照先は同じ outputs/detailed-design/detailed-design.md なので、揺れとしては成立する
  - ただし意味差が小さいため参考扱いに留める
- 推奨
  - 詳細設計書ファイル に統一

## 正規化案まとめ

| 対象 | 推奨正規形 |
| --- | --- |
| implementation-summary.md の呼称 | 実装要約ファイル |
| 手戻りの運用語 | 差し戻し |
| Feature test の表記 | Featureテスト |
| Unit test の表記 | Unitテスト |
| test-result.md の呼称 | テスト結果ファイル |
| case-id 欄名 | 対象 case-id |
| agent 成果物セクション名 | 本エージェントの成果物 |
| agent スキル見出し | # 本エージェントで使用するスキル |

## 除外した候補

- review-findings.appendix.md と review-findings-appendix-template.md
  - 生成物とテンプレートで別オブジェクトのため、同一対象の表記ゆれとは見なさなかった
- test-failures.appendix.md と test-failures-appendix-template.md
  - 上と同じ理由で除外した
- frontmatter の name と実ファイル名の対応
  - cross-reference 観点では大きな不整合は見つからなかった
