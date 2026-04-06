# Copilot 開発フローの問題整理

## 調査方法

- .github 配下の agents、instructions、skills、docs/templates、workflow-artifacts/cases/_case-template を走査した。
- runSubagent を 10 件並列実行し、全体フロー、テスト仕様分岐、テスト実施ループ、命名整合、テンプレート整合、レビュー観点を相互確認した。
- 以下は定義ファイルから読み取れる内容だけを根拠に整理している。実行時 UI の暗黙動作は前提にしていない。

## 実フローのシミュレーション

1. [agents/01_basic-design-import.agent.md](agents/01_basic-design-import.agent.md) が CSV を案件配下へ取り込み、基本設計書と handoff を作成する。
2. [agents/02_detailed-design.agent.md](agents/02_detailed-design.agent.md) が基本設計書をもとに詳細設計書と handoff を作成する。
3. [agents/03_implementation.agent.md](agents/03_implementation.agent.md) が詳細設計書を必須入力として実装し、実装サマリ、変更一覧、handoff を作成する。
4. [agents/04_test-spec.agent.md](agents/04_test-spec.agent.md) が Browser、Feature、Unit からユーザー選択されたテスト種別だけを仕様書化し、[agents/04-01_test-spec-browser.agent.md](agents/04-01_test-spec-browser.agent.md)、[agents/04-02_test-spec-feature.agent.md](agents/04-02_test-spec-feature.agent.md)、[agents/04-03_test-spec-unit.agent.md](agents/04-03_test-spec-unit.agent.md) を必要なものだけ並列利用する。
5. [agents/05_test-exec.agent.md](agents/05_test-exec.agent.md) が test-specification-to-test-execution.md に書かれた選択済みテスト種別だけを実施し、成功ならレビューへ、失敗ならテスト後修正へ進める。
6. [agents/06_post-test-fix.agent.md](agents/06_post-test-fix.agent.md) が失敗原因をもとに修正し、source-change-02.md と post-test-fix-analysis.md を作成する。
7. [agents/07_review-code.agent.md](agents/07_review-code.agent.md) が設計整合、規約、Laravel 責務配置、回帰、セキュリティ、テスト十分性を観点別にレビューし、必要に応じて再テスト用 handoff を返す。

想定される大枠の流れは次の通り。

1. 01 基本設計取込 → 02 詳細設計 → 03 実装 → 04 テスト仕様作成 → 05 テスト実施
2. 05 で失敗あり → 06 テスト後修正 → 07 コードレビュー
3. 05 で失敗なし → 07 コードレビュー
4. 07 で追加確認あり → 05 へ戻って再テスト

## 確認できた条件分岐

1. [agents/03_implementation.agent.md](agents/03_implementation.agent.md) は詳細設計書が存在しない、空である、または読み込めない場合に停止する。
2. [agents/04_test-spec.agent.md](agents/04_test-spec.agent.md) は Browser、Feature、Unit のうち最低 1 つの選択を必須としている。未選択なら停止する。
3. [agents/05_test-exec.agent.md](agents/05_test-exec.agent.md) はテスト失敗の有無で 06 と 07 に分岐する。
4. [agents/07_review-code.agent.md](agents/07_review-code.agent.md) は review-result.md と code-review-to-test-execution.md を分離する思想を持つが、実際の分岐は 05 への handoff しか明示していない。

## 未定義または曖昧な点

1. レビュー完了でフローを閉じる条件が定義されていない。review-result.md には修正要否と再テスト要否の欄がある一方で、[agents/07_review-code.agent.md](agents/07_review-code.agent.md) の handoff は 05 への再テストしかない。
2. レビューで修正が必要と判定されたときに、06 へ戻すのか 03 へ戻すのかが定義されていない。レビュー結果テンプレート側にも戻し先の規則がない。根拠は [agents/07_review-code.agent.md](agents/07_review-code.agent.md) と [docs/templates/review-result-template.md](docs/templates/review-result-template.md)。
3. 06 の修正後に必ず 05 で再テストするのかが定義されていない。[agents/06_post-test-fix.agent.md](agents/06_post-test-fix.agent.md) は 07 に直送している。
4. 3 周目以降の修正と再テストの世代管理が未定義である。変更一覧は source-change-01.md と source-change-02.md に固定され、テスト結果も単一の test-result.md を更新し続ける設計になっている。根拠は [agents/03_implementation.agent.md](agents/03_implementation.agent.md)、[agents/05_test-exec.agent.md](agents/05_test-exec.agent.md)、[agents/06_post-test-fix.agent.md](agents/06_post-test-fix.agent.md)。
5. test-spec のサブエージェント出力形式と review のサブエージェント出力統合規則が未定義である。親エージェントが統合すると書かれているが、返却粒度や競合解決規則がない。根拠は [agents/04_test-spec.agent.md](agents/04_test-spec.agent.md) と [agents/07_review-code.agent.md](agents/07_review-code.agent.md)。
6. case-manifest.md を入力状況と成果物状況の変化まで追跡する運用になっているが、その欄がテンプレートにない。根拠は [instructions/workflow-execution.instructions.md](instructions/workflow-execution.instructions.md)、[skills/workflow--common-artifact-location/SKILL.md](skills/workflow--common-artifact-location/SKILL.md)、[workflow-artifacts/cases/_case-template/case-manifest.md](workflow-artifacts/cases/_case-template/case-manifest.md)。
7. レビュー後再テスト用 handoff に必要な粒度が汎用テンプレートだけでは足りない。対象テスト種別、対象指摘、前提データ、実施コマンド、必要証跡の専用欄がない。根拠は [docs/templates/handoff-template.md](docs/templates/handoff-template.md)、[agents/05_test-exec.agent.md](agents/05_test-exec.agent.md)、[agents/07_review-code.agent.md](agents/07_review-code.agent.md)。

## 課題一覧

1. Critical: 必須スキル参照名が実在スキル名と体系的に一致していない。

影響: agents 側は workflow__ 系で必須スキルを参照しているが、skills 側の frontmatter name は workflow-- 系で統一されている。必須スキルを前提にした運用が定義通りに解決されない可能性が高い。

根拠: [agents/01_basic-design-import.agent.md](agents/01_basic-design-import.agent.md)、[agents/02_detailed-design.agent.md](agents/02_detailed-design.agent.md)、[agents/04_test-spec.agent.md](agents/04_test-spec.agent.md)、[agents/05_test-exec.agent.md](agents/05_test-exec.agent.md)、[agents/07_review-code.agent.md](agents/07_review-code.agent.md)、[skills/workflow--basic-design-fidelity/SKILL.md](skills/workflow--basic-design-fidelity/SKILL.md)、[skills/workflow--review-code-scope/SKILL.md](skills/workflow--review-code-scope/SKILL.md)

2. Critical: テスト成果物の配置先が outputs/test と outputs/testing で衝突している。

影響: 共通配置ルールに従うとテスト成果物が outputs/test に置かれ、テスト仕様作成、テスト実施、レビューの各エージェントは outputs/testing を読みに行く。後工程の入力連鎖が切れる。

根拠: [skills/workflow--common-artifact-location/SKILL.md](skills/workflow--common-artifact-location/SKILL.md)、[agents/04_test-spec.agent.md](agents/04_test-spec.agent.md)、[agents/05_test-exec.agent.md](agents/05_test-exec.agent.md)、[agents/06_post-test-fix.agent.md](agents/06_post-test-fix.agent.md)、[agents/07_review-code.agent.md](agents/07_review-code.agent.md)

3. Critical: テスト失敗後の修正結果が再テストされないままレビューに入る分岐がある。

影響: 05 で失敗したものを 06 で直したあと、その修正が有効だったかを 05 で確認しないまま 07 に進めてしまう。品質確認ループの中心である再テストが抜ける。

根拠: [agents/05_test-exec.agent.md](agents/05_test-exec.agent.md)、[agents/06_post-test-fix.agent.md](agents/06_post-test-fix.agent.md)、[instructions/quality-loop.instructions.md](instructions/quality-loop.instructions.md)

4. Critical: レビュー工程に完了分岐と修正差し戻し分岐がない。

影響: 07 は再テストへの handoff しか持たないため、承認時の正式終了も、修正必須時の正式な戻し先も定義できていない。review-result.md の修正要否と再テスト要否がフローに結びつかない。

根拠: [agents/07_review-code.agent.md](agents/07_review-code.agent.md)、[docs/templates/review-result-template.md](docs/templates/review-result-template.md)

5. High: 後半工程の handoff prompt が self-contained ではなく、正式入力を落としている。

影響: 03 から 04 への prompt は implementation-to-test-specification.md を渡していない。05 から 06、06 から 07 の prompt は case 配下の絶対パスではなくファイル名だけを渡している。複数案件や再実行時に誤読込しやすい。

根拠: [agents/03_implementation.agent.md](agents/03_implementation.agent.md)、[agents/04_test-spec.agent.md](agents/04_test-spec.agent.md)、[agents/05_test-exec.agent.md](agents/05_test-exec.agent.md)、[agents/06_post-test-fix.agent.md](agents/06_post-test-fix.agent.md)、[skills/workflow--common-handoff-format/SKILL.md](skills/workflow--common-handoff-format/SKILL.md)

6. High: テスト種別の選択が初回のユーザー判断に固定され、レビュー結果で新しい test type を追加しにくい。

影響: 04 は選択された test type だけを handoff に載せ、05 もその選択済み test type だけを実施対象にする。レビューで新たに Browser、Feature、Unit の別観点が必要になっても、フロー上は自然に広げられない。

根拠: [agents/04_test-spec.agent.md](agents/04_test-spec.agent.md)、[agents/05_test-exec.agent.md](agents/05_test-exec.agent.md)、[agents/07_review-code.agent.md](agents/07_review-code.agent.md), [instructions/quality-loop.instructions.md](instructions/quality-loop.instructions.md)

7. Medium: 宣言されている成果物に対してテンプレートが不足している。

影響: implementation-summary.md、source-change-01.md、source-change-02.md、post-test-fix-analysis.md、review-findings.appendix.md、test-failures.appendix.md の標準章立てがなく、工程ごとの出力がばらつく。

根拠: [agents/03_implementation.agent.md](agents/03_implementation.agent.md)、[agents/05_test-exec.agent.md](agents/05_test-exec.agent.md)、[agents/06_post-test-fix.agent.md](agents/06_post-test-fix.agent.md)、[agents/07_review-code.agent.md](agents/07_review-code.agent.md)、[docs/templates](docs/templates)

8. Medium: case-manifest の運用要求とテンプレートのスキーマが一致していない。

影響: workflow-execution と common-artifact-location は現在工程だけでなく入力状況と成果物状況も反映対象にしているが、テンプレートは案件情報と現在工程しか持たない。更新要否の判断が属人的になる。

根拠: [instructions/workflow-execution.instructions.md](instructions/workflow-execution.instructions.md)、[skills/workflow--common-artifact-location/SKILL.md](skills/workflow--common-artifact-location/SKILL.md)、[workflow-artifacts/cases/_case-template/case-manifest.md](workflow-artifacts/cases/_case-template/case-manifest.md)

9. Medium: 1 工程 1 正式ファイルの原則と appendix 前提の実装が衝突している。

影響: workflow-core は 1 工程 1 正式ファイルを原則化している一方で、テスト工程とレビュー工程は appendix を正式出力として要求している。成果物設計の原則が二重化している。

根拠: [instructions/workflow-core.instructions.md](instructions/workflow-core.instructions.md)、[agents/05_test-exec.agent.md](agents/05_test-exec.agent.md)、[agents/07_review-code.agent.md](agents/07_review-code.agent.md)、[docs/templates/test-result-template.md](docs/templates/test-result-template.md)、[docs/templates/review-result-template.md](docs/templates/review-result-template.md)

10. Low: source-manifest テンプレートの配置先表記が他ルールとそろっていない。

影響: source-manifest-template は cases 配下から始まる相対表記で、他のルールは .github/workflow-artifacts/cases 配下を正規位置としている。テンプレート単独参照時に誤配置しやすい。

根拠: [docs/templates/source-manifest-template.md](docs/templates/source-manifest-template.md)、[instructions/case-directory.instructions.md](instructions/case-directory.instructions.md)、[skills/workflow--common-artifact-location/SKILL.md](skills/workflow--common-artifact-location/SKILL.md)

## 優先度順の対処案

1. スキル名を workflow-- 系に一本化し、agents 側の必須スキル参照を全面修正する。
2. テスト成果物ディレクトリ名を outputs/testing に統一し、skills、agents、templates の表記差を除去する。
3. 07 の分岐を 明示的な完了、再テスト、修正差し戻し に分け、06 の後に必ず 05 を通す再テスト経路を追加する。
4. 後半工程の handoff prompt を case-id 付きの完全パスに統一し、正式入力の欠落をなくす。
5. case-manifest、review 再テスト用 handoff、不足テンプレート群を追加して、運用判断をテンプレートへ落とす。
