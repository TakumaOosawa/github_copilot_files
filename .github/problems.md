# .github フロー問題整理

## 文書情報
- 文書名: .github フロー問題整理
- 対象: .github 配下の GitHub Copilot 活用フロー一式
- 作成日: 2026-04-06
- 調査方法: runSubagent を 10 本並列実行し、agents、instructions、skills、templates、workflow-artifacts の一次ファイルで再確認した。

## 実作業フローのシミュレーション

### 1. 開始前提
- 実運用の開始点は、既存 case-id を再利用するか、新規案件として case ディレクトリを初期化するかの分岐になる。新規案件では case-manifest、source-manifest、sources、outputs、handoffs を先に用意するルールになっているが、開始 agent の [01_basic-design-import.agent.md](.github/agents/01_basic-design-import.agent.md#L29-L33) は既に case-manifest、source-manifest、CSV が存在する前提で動く。初期化作業は [workflow--common-artifact-location/SKILL.md](.github/skills/workflow--common-artifact-location/SKILL.md#L12-L12)、[workflow--common-artifact-location/SKILL.md](.github/skills/workflow--common-artifact-location/SKILL.md#L40-L43) にルールとしてはあるが、工程としては独立定義されていない。

### 2. 正常系
1. 基本設計取り込み: [01_basic-design-import.agent.md](.github/agents/01_basic-design-import.agent.md) が CSV 群を Markdown の基本設計書へ変換し、basic-design-to-detailed-design を作る。
2. 詳細設計: [02_detailed-design.agent.md](.github/agents/02_detailed-design.agent.md) が基本設計書をもとに detailed-design と detailed-design-to-implementation を作る。
3. 実装: [03_implementation.agent.md](.github/agents/03_implementation.agent.md) が詳細設計を主基準に実装し、implementation-summary、source-change-01、implementation-to-test-specification を作る。
4. テスト仕様: [04_test-spec.agent.md](.github/agents/04_test-spec.agent.md) が Browser、Feature、Unit のうち対象テスト種別を決めて test-spec-*.md と test-specification-to-test-execution を作る。
5. テスト実施: [05_test-exec.agent.md](.github/agents/05_test-exec.agent.md) が test-specification-to-test-execution に従ってテストを実行し、test-result を作る。
6. レビュー: テスト失敗がなければ [07_review-code.agent.md](.github/agents/07_review-code.agent.md) が review-result を作り、完了または差戻しを判定する。

### 3. 条件分岐
- 04 は初回実行時とレビュー差戻し時で分岐する。初回はユーザーに test type を聞き、レビュー差戻し時は code-review-to-test-specification に書かれた追加対象を優先する。[04_test-spec.agent.md](.github/agents/04_test-spec.agent.md#L56-L61)
- 05 はテスト失敗ありなら 06 へ、失敗なしなら 07 へ進む。[05_test-exec.agent.md](.github/agents/05_test-exec.agent.md#L54-L57)、[05_test-exec.agent.md](.github/agents/05_test-exec.agent.md#L71-L72)
- 06 は修正後にレビューへ直行せず、必ず 05 に戻して再テストする。[06_post-test-fix.agent.md](.github/agents/06_post-test-fix.agent.md#L17-L22)
- 07 は判定に応じて 03、04、05、完了の 4 分岐を持つ。修正要なら 03、仕様追加が必要なら 04、既存仕様で再確認できるなら 05、問題なければ完了である。[07_review-code.agent.md](.github/agents/07_review-code.agent.md#L87-L95)

### 4. 未定義または推測運用になっている箇所
- 新規案件起票はルール記述に留まり、実行可能な専用工程がない。
- 07 から 03 へ戻した再実装で、03 がどの追加入力を読むべきかが閉じていない。
- 2 回目以降の post-test-fix をどのファイル名で管理するかが定義されていない。
- 古い handoff をいつ無効化するかの明示ルールが弱く、再入場が多い案件では stale な指示混入の余地がある。

## 課題

### High 1. 05 から 06 への入力契約が矛盾している
05 は test-failures.appendix を「必要な場合のみ」の補助明細として扱っているが、06 への handoff prompt と 06 の正式入力は常にそのファイルを前提にしている。これにより、失敗内容が test-result 本体に収まる運用でも 06 に進めなくなる。

- 根拠: [05_test-exec.agent.md](.github/agents/05_test-exec.agent.md#L11-L11)、[05_test-exec.agent.md](.github/agents/05_test-exec.agent.md#L51-L55)、[05_test-exec.agent.md](.github/agents/05_test-exec.agent.md#L70-L71)、[06_post-test-fix.agent.md](.github/agents/06_post-test-fix.agent.md#L39-L41)、[test-result-template.md](.github/docs/templates/test-result-template.md#L46-L46)
- 影響: テスト失敗時の正規ループが、成果物の粒度次第で実行不能になる。
- 推奨: 05、06、test-result-template の 3 か所で appendix の必須性を一本化する。

### High 2. 07 から 03 への差戻しが受け側で閉じていない
07 は code-review-to-implementation を作って 03 に差し戻すが、03 の正式入力と開始条件は detailed-design-to-implementation と詳細設計書を前提にしており、review-result や code-review-to-implementation を受ける契約がない。

- 根拠: [07_review-code.agent.md](.github/agents/07_review-code.agent.md#L12-L12)、[07_review-code.agent.md](.github/agents/07_review-code.agent.md#L67-L67)、[07_review-code.agent.md](.github/agents/07_review-code.agent.md#L87-L91)、[03_implementation.agent.md](.github/agents/03_implementation.agent.md#L34-L44)
- 影響: レビュー差戻し後の再実装で、何を正式入力として扱うべきかが曖昧になる。
- 推奨: 03 の正式入力に review-result と code-review-to-implementation を追加し、再入場条件を明文化する。

### High 3. 06 が必須スキルの前提入力を満たしていない
06 は workflow--implementation-authority の利用を必須化している一方、その skill が要求する詳細設計書と実装向け handoff を 06 の正式入力に含めていない。03 では同 skill の前提を守る停止条件があるが、06 には同等の防御線がない。

- 根拠: [06_post-test-fix.agent.md](.github/agents/06_post-test-fix.agent.md#L26-L26)、[06_post-test-fix.agent.md](.github/agents/06_post-test-fix.agent.md#L32-L41)、[workflow--implementation-authority/SKILL.md](.github/skills/workflow--implementation-authority/SKILL.md#L8-L9)、[03_implementation.agent.md](.github/agents/03_implementation.agent.md#L42-L44)
- 影響: 設計未確認のままテスト後修正に進めてしまい、修正の根拠が弱くなる。
- 推奨: 06 の正式入力に detailed-design と実装向け handoff を追加するか、06 専用の実装修正ルールを別 skill として定義する。

### Medium 4. 07 はテスト十分性を判定するのにテスト仕様書を正式入力に持たない
07-06 の補助 agent は「テスト仕様とテスト結果」の十分性確認を役割としているが、07 の正式入力には test-result と test-execution-to-code-review はあるものの、test-spec-browser、test-spec-feature、test-spec-unit が含まれていない。これでは、設計した観点と実施結果の突合が一次資料なしになる。

- 根拠: [07-06_review-test-adequacy.agent.md](.github/agents/07-06_review-test-adequacy.agent.md#L3-L13)、[07_review-code.agent.md](.github/agents/07_review-code.agent.md#L49-L56)、[07_review-code.agent.md](.github/agents/07_review-code.agent.md#L81-L85)
- 影響: テスト十分性レビューが、実際には test-result 中の要約に依存した表面的確認に寄りやすい。
- 推奨: 07 の正式入力に対象 test-spec-*.md を追加し、05 から 07 への handoff にも参照仕様書を明記させる。

### Medium 5. 新規案件起票が実行可能な工程として定義されていない
新規案件では _case-template をもとに case ディレクトリ一式を作る必要があるが、開始 agent の 01 は既に case-manifest、source-manifest、CSV が存在する前提で始まる。つまり、新規案件の初期化は workflow ではなく運用者の暗黙作業になっている。

- 根拠: [01_basic-design-import.agent.md](.github/agents/01_basic-design-import.agent.md#L29-L33)、[workflow--common-artifact-location/SKILL.md](.github/skills/workflow--common-artifact-location/SKILL.md#L12-L12)、[workflow--common-artifact-location/SKILL.md](.github/skills/workflow--common-artifact-location/SKILL.md#L40-L43), [case-manifest.md](.github/workflow-artifacts/cases/_case-template/case-manifest.md)
- 影響: 「案件を 0 から開始する」操作が一続きのフローとして完結しない。
- 推奨: case 作成専用の 00 系 agent を追加するか、01 の開始条件に初期化手順を明文化する。

### Medium 6. 04 と 05 では case-manifest 更新責務が抜けやすい
artifact 配置ルールは毎工程の毎操作で case-manifest の更新要否を検討するよう求めているが、04 と 05 の正式入力一覧には case-manifest が入っていない。testing 工程は成果物と handoff が大きく動くため、むしろ drift が起きやすい。

- 根拠: [workflow--common-artifact-location/SKILL.md](.github/skills/workflow--common-artifact-location/SKILL.md#L43-L43)、[04_test-spec.agent.md](.github/agents/04_test-spec.agent.md#L31-L49)、[05_test-exec.agent.md](.github/agents/05_test-exec.agent.md#L31-L43)
- 影響: 現在工程、直近 handoff、テスト成果物状況が manifest と実態でずれやすい。
- 推奨: 04 と 05 の正式入力に case-manifest を追加し、更新要否確認を開始条件へ明記する。

### Medium 7. 複数回の再修正ループで履歴管理契約が閉じていない
03 は handoff では source-change-*.md を読ませる一方、正式成果物は source-change-01 固定である。06 も source-change-02 と post-test-fix-analysis 固定で、07 もその 2 本だけを主入力として優先する。2 回目以降の post-test-fix や再レビューで履歴をどう残すかが定義されていない。

- 根拠: [03_implementation.agent.md](.github/agents/03_implementation.agent.md#L11-L11)、[03_implementation.agent.md](.github/agents/03_implementation.agent.md#L50-L52)、[06_post-test-fix.agent.md](.github/agents/06_post-test-fix.agent.md#L46-L48)、[07_review-code.agent.md](.github/agents/07_review-code.agent.md#L55-L56)、[07_review-code.agent.md](.github/agents/07_review-code.agent.md#L82-L82)、[test-result-template.md](.github/docs/templates/test-result-template.md#L8-L8)
- 影響: 上書き前提になりやすく、複数回の修正理由と再テスト経緯を後から追いにくい。
- 推奨: source-change-N と post-test-fix-analysis-N の連番方針、または appendix による履歴保持方針を明文化する。

### Medium 8. 04 のテスト種別選択が対話依存で、非対象理由の記録契約が弱い
04 は初回開始時に Browser、Feature、Unit の対象をユーザーに質問して決めるが、上流の implementation-summary では重点確認観点や条件分岐までは持てても、非対象にした test type の理由までは構造化していない。結果として、未選択が意図的除外か見落としかを後工程が判別しにくい。

- 根拠: [04_test-spec.agent.md](.github/agents/04_test-spec.agent.md#L57-L61)、[04_test-spec.agent.md](.github/agents/04_test-spec.agent.md#L88-L88)、[implementation-summary-template.md](.github/docs/templates/implementation-summary-template.md#L32-L34)
- 影響: テスト観点の取りこぼしが、人手の質問応答品質に依存する。
- 推奨: implementation-to-test-specification または implementation-summary に、対象 test type と非対象理由を必須項目として追加する。

### Low 9. 説明文と参照文字列の精度不足が discovery を下げている
実行不能ではないが、agent や skill の description、参照先表現にズレがあり、必要な定義を見つけにくくしている。

- workflow--basic-design-fidelity はシート順を case-manifest に従うとしているが、順序管理の実体は source-manifest 側のテンプレートにある。[workflow--basic-design-fidelity/SKILL.md](.github/skills/workflow--basic-design-fidelity/SKILL.md#L12-L12)、[source-manifest-template.md](.github/docs/templates/source-manifest-template.md#L12-L13)、[source-manifest-template.md](.github/docs/templates/source-manifest-template.md#L33-L33)
- workflow--test-spec-viewpoint の description は「ブラウザテストと PHPUnit テスト」の 2 分類に読めるが、実フローは Browser、Feature、Unit の 3 分割で運用している。[workflow--test-spec-viewpoint/SKILL.md](.github/skills/workflow--test-spec-viewpoint/SKILL.md#L3-L3)、[04_test-spec.agent.md](.github/agents/04_test-spec.agent.md#L4-L4)、[04_test-spec.agent.md](.github/agents/04_test-spec.agent.md#L57-L57)
- workflow--review-code-scope の description はテスト十分性を省略しているが、本文と親 agent は含めている。[workflow--review-code-scope/SKILL.md](.github/skills/workflow--review-code-scope/SKILL.md#L3-L7)、[07_review-code.agent.md](.github/agents/07_review-code.agent.md#L3-L3)
- 07-03_review-laravel-structure の description は Model と View を省略しているが、本文では対象に入っている。[07-03_review-laravel-structure.agent.md](.github/agents/07-03_review-laravel-structure.agent.md#L3-L14)
- workflow--common-design-template-guide は docs/templates とだけ書いており、他定義の .github/docs/templates という表現と揃っていない。[workflow--common-design-template-guide/SKILL.md](.github/skills/workflow--common-design-template-guide/SKILL.md#L10-L10)

## 総括
- 正常系の一本線は定義されているが、実際に運用で詰まりやすいのは新規案件起票と差戻しループである。
- 最優先で直すべきなのは、05 と 06 の appendix 契約、07 から 03 への再実装契約、06 の正式入力不足の 3 点である。
- その次に、07 のテスト仕様書参照、testing 工程の case-manifest 更新責務、再修正履歴の保持方針を固めると、フロー全体がかなり閉じる。
