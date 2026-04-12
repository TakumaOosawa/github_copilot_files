# .github ワークフロー問題整理

## 文書情報
- 文書名: .github ワークフロー問題整理
- 対象: .github 配下の GitHub Copilot 活用フロー定義
- 作成日: 2026-04-07
- 更新日: 2026-04-12
- 調査方法: 静的読解 + 構造整理後の整合確認
- サブエージェント内訳: Explore x4、06-01_review-coding-rules、06-02_review-design-alignment、06-03_review-laravel-structure、06-04_review-regression、06-05_review-security、06-06_review-test-adequacy
- 対象範囲:
  - .github/agents
  - .github/instructions
  - .github/skills
  - .github/docs/templates
  - .github/docs/templates/_case-template

## 結論
- 親工程は 00_case-init から 06_review-code までの 7 工程に整理した。
- テスト失敗時の修正は独立工程を立てず、03_implementation への差し戻しとして扱う構造に統一した。
- 05_test-exec の失敗時 handoff は test-execution-to-implementation.md に、成功時 handoff は test-execution-to-code-review.md に整理した。
- レビュー親エージェントとレビュー補助エージェントは 06 系の番号へそろえ、親工程番号と補助番号の関係が追いやすくなった。
- 一方で、テスト実施中に test-spec 不足が判明した場合の正式な戻し先、test-result と test-spec のトレーサビリティ、完了時の case-manifest 更新など、構造以外の運用課題は残っている。

## 想定される実作業フロー

### 主経路
1. 00_case-init
2. 01_basic-design-import
3. 02_detailed-design
4. 03_implementation
5. 04_test-spec
6. 05_test-exec
7. 06_review-code
8. code-review-complete で終了

### 主な条件分岐
| 起点工程 | 条件 | 遷移先 | 正式 handoff |
| --- | --- | --- | --- |
| 05_test-exec | テスト失敗あり | 03_implementation | test-execution-to-implementation.md |
| 05_test-exec | テスト失敗なし | 06_review-code | test-execution-to-code-review.md |
| 06_review-code | 修正要 | 03_implementation | code-review-to-implementation.md |
| 06_review-code | 修正不要かつ新規テスト種別または仕様拡張が必要 | 04_test-spec | code-review-to-test-specification.md |
| 06_review-code | 修正不要かつ既存仕様で再確認可能 | 05_test-exec | code-review-to-test-execution.md |
| 06_review-code | 修正不要かつ再テスト不要 | 終了 | code-review-complete.md |

### 実際にシミュレーションしたシナリオ
1. 正常系: 00 → 01 → 02 → 03 → 04 → 05 成功 → 06 承認 → 完了
2. テスト失敗系: 00 → 01 → 02 → 03 → 04 → 05 失敗 → 03 再入場 → 04 → 05 再テスト → 06
3. レビュー差戻し系: 06 で修正要 → 03 再入場 → 04 → 05 → 06
4. レビュー起因のテスト仕様追加系: 06 で仕様追加要 → 04 再入場 → 05 → 06
5. レビュー起因の再テスト系: 06 で既存仕様の再確認要 → 05 再入場 → 06

## シミュレーション結果

### 1. 正常系
- 主経路は成立している。
- 00 から 04 までは、入力と成果物のつながりが比較的素直で追いやすい。
- 05 成功時に 06 へ進み、06 で code-review-complete.md を出す終端も定義されている。
- ただし、完了時に case-manifest をどう最終状態へ更新するかは依然として明示が弱い。

### 2. テスト失敗系
- 05 失敗時に 03 へ戻す構造にしたことで、親工程の並びは 00 から 06 のまま維持できる。
- 03 は detailed-design.md と detailed-design-to-implementation.md を主基準としつつ、test-result.md と test-execution-to-implementation.md を併読して修正方針を判断するため、失敗対応時の入力契約が一本化された。
- 失敗起点の修正内容は source-change-N.md と implementation-summary.md に集約し、必要な場合は test-execution-fix-analysis-N.md に失敗分析を残せる。
- 一方で、05 実行中に「実装ではなく test-spec の不足」と判断した場合の正式な 04 戻しは依然として未定義である。

### 3. レビュー差戻し系
- 06 が code-review-to-implementation.md で 03 に戻す流れは維持されている。
- 03 は review-result.md と code-review-to-implementation.md を読むよう要求しているため、レビュー指摘を見落とさずに修正へ入る設計になっている。
- ただし、06 が持っていた再テスト観点を 03 が implementation-to-test-specification.md に必ず再掲する契約は、なお明文化を強める余地がある。

### 4. レビュー起因のテスト仕様追加系
- 06 から 04 に戻す分岐は定義されている。
- 04 は既存の test-specification-to-test-execution.md を参照しつつ追加更新する前提を持つ。
- ただし、以前に「非対象」としたテスト種別をレビューで追加対象に変えるときの優先ルールは依然として曖昧である。

### 5. レビュー起因の再テスト系
- 06 から 05 に戻す分岐は定義されている。
- 05 側の追加入力は code-review-to-test-execution.md に寄せたため、review-result.md と再テスト handoff の責務は整理しやすくなった。
- ただし、test-result.md が参照 test-spec や観点 ID を構造化していないため、十分性確認はまだ手書き運用へ依存している。

## 未定義または曖昧な点
- 05 で「テスト仕様そのものが不十分」と分かった場合に、04 へ正式に戻す経路がない。
- 03 の出力が全テスト種別を「非対象」または「要確認」とした場合、04 は停止条件に入りやすいが、その場合の上位差し戻し先が曖昧である。
- test-result.md は再テスト区分を持てるようにしたが、どの test-spec のどの観点に対応するかを構造化していない。
- 02 と 06 は開始条件セクションを持たず、他工程と比べて case-manifest 更新要否の記述が薄い。
- Laravel 責務分離ルールはあるが、03 と 06 の成果物テンプレートには責務分離を必ず記録する専用欄がない。
- .github/agents/00_case-init.agent.md の「成果物テンプレート」節は、テンプレートと初期記入例を同列に扱っており、参照意図がやや曖昧である。

## 課題一覧

### High

#### H-01. テスト実施からテスト仕様へ戻す正式経路がない
- 対象:
  - .github/agents/05_test-exec.agent.md
  - .github/agents/04_test-spec.agent.md
  - .github/agents/06_review-code.agent.md
- 内容:
  - 05 の正式 handoff は 03 か 06 の二択であり、04 へ戻す経路が無い。
  - 実行中に「実装バグではなく test-spec の不足や誤り」と判断しても、正式な戻し先は定義されていない。
- 影響:
  - テスト仕様の不備を実装修正へ押し込むか、手順外運用で補うしかなくなる。
- 推奨対応:
  - 05 から 04 へ戻す handoff を追加するか、05 が spec defect を検出したときに 04 へ差し替え可能な規則を定義する。

#### H-02. テスト十分性レビューが変更差分を正式入力に持たない
- 対象:
  - .github/agents/06_review-code.agent.md
  - .github/agents/06-06_review-test-adequacy.agent.md
- 内容:
  - 06 は test-result.md と test-execution-to-code-review.md と参照 test-spec の突合を重視している。
  - 06-06 へ渡す入力は test-result と handoff と test-spec 中心で、implementation-summary や source-change が常に明示されていない。
- 影響:
  - 「何が変わったか」に対して十分なテストだったかを定義上は評価しきれない。
- 推奨対応:
  - 06-06 への正式入力に implementation-summary.md と最新の source-change-N.md を追加する。

### Medium

#### M-01. case-manifest 更新ルールの記述が工程ごとに不均一
- 対象:
  - .github/skills/workflow--common-artifact-location/SKILL.md
  - .github/skills/workflow--common-case-manifest-format/SKILL.md
  - .github/agents/01_basic-design-import.agent.md
  - .github/agents/02_detailed-design.agent.md
  - .github/agents/03_implementation.agent.md
  - .github/agents/06_review-code.agent.md
- 内容:
  - case-manifest 関連スキルは毎工程で case-manifest 更新要否を検討するよう求めている。
  - 04 と 05 は開始条件でその旨を明文化しているが、02 と 06 は開始条件自体が無く、01 と 03 も manifest 更新の明示が弱い。
- 影響:
  - 工程が進んでも case-manifest の現在工程、入力状況、成果物状況が揃って更新されない恐れがある。
- 推奨対応:
  - 全工程に開始条件節を揃え、manifest 更新要否の確認を明文化する。

#### M-02. test-result テンプレートに test-spec とのトレーサビリティがない
- 対象:
  - .github/docs/templates/test-result-template.md
  - .github/docs/templates/handoff-template.md
- 内容:
  - test-result-template.md には対象 test-spec や観点 ID を構造化する欄がない。
  - handoff-template.md も汎用的で、テスト仕様との結び付きは利用側の記述に委ねられている。
- 影響:
  - 06 が test-spec と test-result の対応を確実に追うには、毎回手書き運用へ依存する。
- 推奨対応:
  - test-result.md に test-spec ファイル名、観点 ID、再実施対象を入れる欄を追加する。

#### M-03. Laravel 責務分離の要求が 03 と 06 で弱い
- 対象:
  - .github/skills/workflow--common-design-template-guide/SKILL.md
  - .github/skills/workflow--implementation-authority/SKILL.md
  - .github/skills/workflow--review-code-scope/SKILL.md
  - .github/agents/03_implementation.agent.md
  - .github/agents/06_review-code.agent.md
  - .github/docs/templates/implementation-summary-template.md
- 内容:
  - Laravel 責務分離ルールは Route、Middleware、Controller、Request、Service、Repository、Model、View の責務分離を要求している。
  - 02 の詳細設計ではこの粒度を扱えるが、03 と 06 の成果物テンプレートには責務逸脱の確認結果を残す専用欄がない。
- 影響:
  - Laravel 前提の設計整合が、後段では「意識する」止まりになりやすい。
- 推奨対応:
  - implementation-summary.md と review-result.md に責務配置確認欄を追加する。

### Low

#### L-01. 00_case-init のテンプレート参照の表現が曖昧
- 対象:
  - .github/agents/00_case-init.agent.md
- 内容:
  - 「成果物テンプレート」節に、実テンプレートと初期記入例が混在している。
- 影響:
  - 実装者がどれを雛形、どれを参照例として扱うべきか迷う可能性がある。
- 推奨対応:
  - 「テンプレート」と「初期記入例」を別見出しに分ける。

#### L-02. 完了後の後続状態が弱い
- 対象:
  - .github/agents/06_review-code.agent.md
  - .github/docs/templates/case-manifest-template.md
- 内容:
  - code-review-complete.md はあるが、完了後の case-manifest 更新や後続の引継ぎ先は定義されていない。
- 影響:
  - ワークフロー終端は分かるが、案件管理上の完了状態が文書間で揃わない可能性がある。
- 推奨対応:
  - case-manifest の完了状態更新を 06 の終了条件へ追加する。

## 優先対応案
1. 05 から 04 への正式差し戻し条件と handoff を定義する。
2. 06-06_review-test-adequacy へ implementation-summary.md と最新 source-change-N.md を正式入力として明文化する。
3. test-result.md と review/test handoff に、test-spec 名と観点 ID を持たせてトレーサビリティを構造化する。
4. case-manifest の完了状態更新を 06_review-code の終了条件へ追加する。

## 補足
- 本書は .github 配下の定義を静的に読んでシミュレーションした結果であり、実ランタイムでの暗黙補完までは確認していない。
- 2026-04-12 時点では、親工程の構造整理と主要 handoff 名の整合は反映済みである。
