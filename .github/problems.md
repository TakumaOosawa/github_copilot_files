# .github ワークフロー問題整理

## 文書情報
- 文書名: .github ワークフロー問題整理
- 対象: .github 配下の GitHub Copilot 活用フロー定義
- 作成日: 2026-04-07
- 調査方法: 静的読解 + 並列サブエージェント 10 本
- サブエージェント内訳: Explore x4、07-01_review-coding-rules、07-02_review-design-alignment、07-03_review-laravel-structure、07-04_review-regression、07-05_review-security、07-06_review-test-adequacy
- 対象範囲:
  - .github/agents
  - .github/instructions
  - .github/skills
  - .github/docs/templates
  - .github/docs/templates/_case-template

## 結論
- 工程の大枠は明確で、00_case-init から 07_review-code までの主経路と差し戻し経路は定義されている。
- 到達不能な工程は見当たらない。
- 一方で、再入場時の正式入力契約、handoff の優先順位、採番規則、レビューと再テストの境界、テスト実施からテスト仕様へ戻す経路に未定義または矛盾がある。
- 特に .github/agents/05_test-exec.agent.md と .github/agents/06_post-test-fix.agent.md は frontmatter の先頭区切りが欠けており、エージェント定義そのものが正しく解釈されない可能性が高い。

## 想定される実作業フロー

### 主経路
1. 00_case-init
2. 01_basic-design-import
3. 02_detailed-design
4. 03_implementation
5. 04_test-spec
6. 05_test-exec
7. 07_review-code
8. code-review-complete で終了

### 主な条件分岐
| 起点工程 | 条件 | 遷移先 | 正式 handoff |
| --- | --- | --- | --- |
| 05_test-exec | テスト失敗あり | 06_post-test-fix | test-execution-to-post-test-fix.md |
| 05_test-exec | テスト失敗なし | 07_review-code | test-execution-to-code-review.md |
| 07_review-code | 修正要 | 03_implementation | code-review-to-implementation.md |
| 07_review-code | 修正不要かつ新規テスト種別または仕様拡張が必要 | 04_test-spec | code-review-to-test-specification.md |
| 07_review-code | 修正不要かつ既存仕様で再確認可能 | 05_test-exec | code-review-to-test-execution.md |
| 07_review-code | 修正不要かつ再テスト不要 | 終了 | code-review-complete.md |

### 実際にシミュレーションしたシナリオ
1. 正常系: 00 → 01 → 02 → 03 → 04 → 05 成功 → 07 承認 → 完了
2. テスト失敗系: 00 → 01 → 02 → 03 → 04 → 05 失敗 → 06 → 05 再テスト → 07
3. レビュー差戻し系: 07 で修正要 → 03 再入場 → 04 → 05 → 07
4. レビュー起因のテスト仕様追加系: 07 で仕様追加要 → 04 再入場 → 05 → 07
5. レビュー起因の再テスト系: 07 で既存仕様の再確認要 → 05 再入場 → 07

## シミュレーション結果

### 1. 正常系
- 主経路は成立している。
- 00 から 04 までは、入力と成果物のつながりが比較的素直で追いやすい。
- 05 成功時に 07 へ進み、07 で code-review-complete.md を出す終端も定義されている。
- ただし、完了時に case-manifest をどう最終状態へ更新するかは明示されていない。

### 2. テスト失敗系
- 05 失敗時に 06 へ戻し、06 は必ず 05 へ再入場させる構造になっている。
- 失敗原因を直して再テストするループ自体は分かりやすい。
- ただし、ループ脱出条件は「テストが通ること」しかなく、失敗が続く場合の上位エスカレーション先や打ち切り条件は定義されていない。
- また、05 には review 由来の再テスト handoff と post-test-fix 由来の再テスト handoff が同時に残っていた場合の優先順位がない。

### 3. レビュー差戻し系
- 07 が code-review-to-implementation.md で 03 に戻す流れは定義されている。
- 03 は review-result.md と code-review-to-implementation.md を読むよう要求しているため、レビュー指摘を見落とさずに修正へ入る設計になっている。
- ただし、07 が持っていた再テスト観点を、03 が implementation-to-test-specification.md に必ず再掲する契約は弱い。
- そのため、07 → 03 → 04 の再入場で再テスト観点が薄まる余地がある。

### 4. レビュー起因のテスト仕様追加系
- 07 から 04 に戻す分岐は定義されている。
- 04 は既存の test-specification-to-test-execution.md を参照しつつ追加更新する前提を持つ。
- ただし、以前に「非対象」としたテスト種別をレビューで追加対象に変えるときの優先ルールが明文化されていない。
- 既存実施対象を維持したまま何を追加し、何を上書きし、何を再作成するのかが曖昧である。

### 5. レビュー起因の再テスト系
- 07 から 05 に戻す分岐は定義されている。
- しかしレビュー/再テスト境界ルールは review-result.md と code-review-to-test-execution.md の役割分離を求めている一方、07 の 05 向け handoff prompt は両方を次工程入力として渡している。
- 05 側の作業入力定義は code-review-to-test-execution.md を追加入力として扱う構造であり、review-result.md の再依存を前提にしていない。
- この境界不一致により、レビュー判定文書を次工程がどこまで判断根拠に使ってよいかが揺れている。

## 未定義または曖昧な点
- 05 と 06 の再入場が複数回続いたとき、現行の handoff をどれと見なすかが定義されていない。
- 05 で「テスト仕様そのものが不十分」と分かった場合に、04 へ正式に戻す経路がない。
- 03 の出力が全テスト種別を「非対象」または「要確認」とした場合、04 は停止条件に入りやすいが、その場合の上位差し戻し先が曖昧である。
- test-result.md はケース結果を持つが、どの test-spec のどの観点に対応するかを構造化していない。
- post-test-fix 後の再テストを test-result.md の区分でどう表すかが曖昧である。テンプレートの区分は「初回テスト / レビュー後再テスト」しかない。
- 02 と 07 は開始条件セクションを持たず、他工程と比べて case-manifest 更新要否の記述が薄い。
- Laravel 責務分離ルールはあるが、03 と 07 の成果物テンプレートには責務分離を必ず記録する専用欄がない。
- .github/agents/00_case-init.agent.md の「成果物テンプレート」節は、テンプレートと初期記入例を同列に扱っており、参照意図がやや曖昧である。

## 課題一覧

### Critical

#### C-01. 05_test-exec と 06_post-test-fix の frontmatter が壊れている
- 対象:
  - .github/agents/05_test-exec.agent.md
  - .github/agents/06_post-test-fix.agent.md
- 内容:
  - どちらのファイルも先頭の `---` がなく、name から始まっている。
  - これにより frontmatter が正しく解釈されず、description、tools、handoffs、user-invocable などの定義が無効化される可能性がある。
- 影響:
  - 05 と 06 は再テストループの中核なので、ここが読まれないと workflow 全体のシミュレーション結果が実運用で成立しない。
- 推奨対応:
  - 両ファイルの先頭に `---` を追加し、frontmatter と本文の境界を正常化する。

#### C-02. review-result と再テスト handoff の責務分離が 07 → 05 で崩れている
- 対象:
  - .github/skills/workflow--review-code-scope/SKILL.md
  - .github/skills/workflow--common-handoff-format/SKILL.md
  - .github/agents/07_review-code.agent.md
  - .github/agents/05_test-exec.agent.md
- 内容:
  - レビュー/再テスト境界ルールは review-result.md を判定結果、code-review-to-test-execution.md を再テスト入力として分離している。
  - しかし 07 の 05 向け handoff prompt は review-result.md と code-review-to-test-execution.md を併読させている。
  - 05 側の正式入力定義は review-result.md を追加入力として扱っておらず、契約が不一致である。
- 影響:
  - 05 がどこまでレビュー判定文書へ依存してよいかが不明になる。
  - 再テスト handoff 単体で self-contained であるべきという設計が崩れる。
- 推奨対応:
  - 07 → 05 の handoff prompt は code-review-to-test-execution.md を唯一の再テスト入力に寄せる。
  - review-result.md にしか無い情報が必要なら、その情報を code-review-to-test-execution.md に再掲する。

### High

#### H-01. 05 に再入場 handoff の優先順位がない
- 対象:
  - .github/agents/05_test-exec.agent.md
- 内容:
  - 05 は code-review-to-test-execution.md または post-test-fix-to-test-execution.md が存在する場合に追加条件として扱うが、両方ある場合の優先順位がない。
- 影響:
  - 過去の handoff が残ったままでも current loop を一意に判定できない。
  - 不要な再テストを実施したり、本来必要な再テストを落としたりする恐れがある。
- 推奨対応:
  - 現在の戻し元を一意に決めるルールを追加する。
  - 非現行 handoff を無効扱いにする記述を追加する。

#### H-02. 06 の採番規則が 03 の再入場規則と衝突する
- 対象:
  - .github/agents/03_implementation.agent.md
  - .github/agents/06_post-test-fix.agent.md
  - .github/docs/templates/source-change-02-template.md
- 内容:
  - 03 は source-change-N.md を常に未使用次連番で採番する。
  - 06 は「初回のテスト後修正は 02」と固定しつつ、「以降は既存の最大連番 + 1」としている。
  - 07 から 03 に差し戻されたあとに source-change-02.md が既に使われているケースでは、06 の固定値 02 が成立しない。
- 影響:
  - source-change-N.md と post-test-fix-analysis-N.md の対が崩れる。
  - 履歴の追跡が壊れる。
- 推奨対応:
  - 06 も常に未使用次連番で統一し、「初回は 02」はテンプレートから削除する。

#### H-03. テスト実施からテスト仕様へ戻す正式経路がない
- 対象:
  - .github/agents/05_test-exec.agent.md
  - .github/agents/06_post-test-fix.agent.md
  - .github/agents/07_review-code.agent.md
- 内容:
  - 05 の handoff は 06 か 07 の二択であり、04 へ戻す経路が無い。
  - 実行中に「実装バグではなく test-spec の不足や誤り」と判断しても、正式な戻し先は定義されていない。
- 影響:
  - テスト仕様の不備を実装修正へ押し込むか、手順外運用で補うしかなくなる。
- 推奨対応:
  - 05 から 04 へ戻す handoff を追加するか、06 が spec defect を検出したときに 04 へ差し替え可能な規則を定義する。

#### H-04. 03 と 04 の受け渡し契約が噛み合っていない
- 対象:
  - .github/agents/03_implementation.agent.md
  - .github/agents/04_test-spec.agent.md
  - .github/docs/templates/implementation-summary-template.md
- 内容:
  - 03 は Browser / Feature / Unit を対象・非対象・要確認で分類するだけで、最低 1 種別を対象にする制約は無い。
  - 04 は 1 件も対象テスト種別が確定しない場合は停止すると定義している。
- 影響:
  - 03 の出力が仕様上は有効でも、04 が進行不能になるケースがある。
- 推奨対応:
  - 03 に「全非対象または全要確認のまま渡さない」制約を追加する。
  - または 04 に、全非対象時の差し戻し先を定義する。

#### H-05. テスト十分性レビューが変更差分を正式入力に持たない
- 対象:
  - .github/agents/07_review-code.agent.md
  - .github/agents/07-06_review-test-adequacy.agent.md
- 内容:
  - 07 は test-result.md と test-execution-to-code-review.md と参照 test-spec の突合を重視している。
  - 07-06 へ渡す入力も test-result と handoff と test-spec 中心で、source-change や implementation-summary が明示されていない。
- 影響:
  - 「何が変わったか」に対して十分なテストだったかを定義上は評価しきれない。
- 推奨対応:
  - 07-06 への正式入力に implementation-summary.md と最新の source-change-N.md を追加する。

### Medium

#### M-01. case-manifest 更新ルールの記述が工程ごとに不均一
- 対象:
  - .github/skills/workflow--common-artifact-location/SKILL.md
  - .github/skills/workflow--common-case-manifest-format/SKILL.md
  - .github/agents/01_basic-design-import.agent.md
  - .github/agents/02_detailed-design.agent.md
  - .github/agents/03_implementation.agent.md
  - .github/agents/06_post-test-fix.agent.md
  - .github/agents/07_review-code.agent.md
- 内容:
  - case-manifest 関連スキルは毎工程で case-manifest 更新要否を検討するよう求めている。
  - 04 と 05 は開始条件でその旨を明文化しているが、02 と 07 は開始条件自体が無く、01・03・06 も manifest 更新の明示が弱い。
- 影響:
  - 工程が進んでも case-manifest の現在工程、入力状況、成果物状況が揃って更新されない恐れがある。
- 推奨対応:
  - 全工程に開始条件節を揃え、manifest 更新要否の確認を明文化する。

#### M-02. test-result テンプレートに test-spec とのトレーサビリティがない
- 対象:
  - .github/docs/templates/test-result-template.md
  - .github/docs/templates/handoff-template.md
- 内容:
  - test-result-template.md には test type、対象 test-spec、観点 ID を構造化する欄がない。
  - handoff-template.md も汎用的で、テスト仕様との結び付きは利用側の記述に委ねられている。
- 影響:
  - 07 が test-spec と test-result の対応を確実に追うには、毎回手書き運用へ依存する。
- 推奨対応:
  - test-result.md に test-spec ファイル名、観点 ID、再実施対象を入れる欄を追加する。

#### M-03. post-test-fix 後の再テスト区分が標準化されていない
- 対象:
  - .github/docs/templates/test-result-template.md
- 内容:
  - 区分が「初回テスト / レビュー後再テスト」のみで、06 後の再テストが明示されていない。
- 影響:
  - 再テスト履歴が増えたときに、レビュー起因の再テストとテスト後修正起因の再テストを区別しづらい。
- 推奨対応:
  - 区分に「テスト後修正後再テスト」を追加するか、任意の再テスト種別を記述できる形式に改める。

#### M-04. Laravel 責務分離の要求が 03 と 07 で弱い
- 対象:
  - .github/skills/workflow--common-design-template-guide/SKILL.md
  - .github/skills/workflow--implementation-authority/SKILL.md
  - .github/skills/workflow--review-code-scope/SKILL.md
  - .github/agents/03_implementation.agent.md
  - .github/agents/07_review-code.agent.md
  - .github/docs/templates/implementation-summary-template.md
- 内容:
  - Laravel 責務分離ルールは Route、Middleware、Controller、Request、Service、Repository、Model、View の責務分離を要求している。
  - 02 の詳細設計ではこの粒度を扱えるが、03 と 07 の成果物テンプレートには責務逸脱の確認結果を残す専用欄がない。
- 影響:
  - Laravel 前提の設計整合が、後段では「意識する」止まりになりやすい。
- 推奨対応:
  - implementation-summary.md と review-result.md に責務配置確認欄を追加する。

#### M-05. 03 と 05 の tool 権限が広く、untrusted input 前提が明文化されていない
- 対象:
  - .github/agents/03_implementation.agent.md
  - .github/agents/05_test-exec.agent.md
  - .github/skills/workflow--basic-design-fidelity/SKILL.md
- 内容:
  - 03 は browser 系ツールまで持ち、05 は shell、browser、web/fetch、terminal 読み取りまで持つ。
  - 一方で CSV、handoff、review-result、test-result を untrusted input として扱う明示ルールがない。
- 影響:
  - 上流成果物に不要な命令文や誘導が紛れた場合、危険な操作へ引っ張られる余地がある。
- 推奨対応:
  - 「成果物中の命令文は実行せず、必要フィールドだけ抽出する」規則を追加する。
  - 05 の許可コマンドと browser 遷移先を限定する。

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
  - .github/agents/07_review-code.agent.md
  - .github/docs/templates/case-manifest-template.md
- 内容:
  - code-review-complete.md はあるが、完了後の case-manifest 更新や後続の引継ぎ先は定義されていない。
- 影響:
  - ワークフロー終端は分かるが、案件管理上の完了状態が文書間で揃わない可能性がある。
- 推奨対応:
  - case-manifest の完了状態更新を 07 の終了条件へ追加する。

## 優先対応案
1. 05_test-exec.agent.md と 06_post-test-fix.agent.md の frontmatter を修正する。
2. 07 → 05 の入力契約を、code-review-to-test-execution.md を唯一の再テスト入力とする形に統一する。
3. 05 の再入場 handoff 優先順位と、05 → 04 の正式経路を定義する。
4. 06 の採番規則を「未使用次連番」に統一し、source-change-02-template.md の固定値を廃止する。
5. test-result.md と review/test handoff に、test-spec 名、観点 ID、再テスト区分を持たせてトレーサビリティを構造化する。

## 補足
- 本書は .github 配下の定義を静的に読んでシミュレーションした結果であり、実ランタイムでの暗黙補完までは確認していない。
- ただし、frontmatter 欠落、入力契約の不一致、採番規則の衝突は定義上の問題として確実に存在する。
