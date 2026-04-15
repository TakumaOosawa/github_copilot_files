# .github 配下 ソースコード修正対応の課題整理

## 調査結論

- 現行構成でも、詳細設計と handoff がそろった通常の実装変更は [agents/03_implementation.agent.md](agents/03_implementation.agent.md) で対応できる。実際に `edit/editFiles` が付与されており、[skills/workflow--implementation-authority/SKILL.md](skills/workflow--implementation-authority/SKILL.md) でもコード探索とコード編集を前提にしている。
- ただし現状の定義は [workflow-artifacts](workflow-artifacts) 配下の成果物更新に重心があり、現行ソースコードの直接調査、失敗原因の切り分け、緊急修正、修正後の検証に加えて、差分案件でも周辺の既存実装を参照基準として明示するための規約が不足している。
- そのため、まずは既存エージェントとスキルの修正が必要である。さらに、設計書が十分でない不具合対応やホットフィックスまで対象にするなら、新規カスタムエージェントの追加が妥当である。

## 現状で対応できる範囲

- [agents/03_implementation.agent.md](agents/03_implementation.agent.md) は、詳細設計と各種 handoff を正式入力にして、実装と成果物更新を行う主担当になっている。
- [agents/02_detailed-design.agent.md](agents/02_detailed-design.agent.md) と [agents/03_implementation.agent.md](agents/03_implementation.agent.md) の定義からは、1 つの case-id につき `outputs/detailed-design/detailed-design.md` を current な詳細設計書として更新し続け、履歴は `source-change-N.md` や handoff 側で持つ運用が読み取れる。
- [agents/05_test-exec.agent.md](agents/05_test-exec.agent.md) は、テスト実施と失敗時の差し戻しを担っており、[docs/templates/test-execution-fix-analysis-template.md](docs/templates/test-execution-fix-analysis-template.md) で失敗分析の記録先も用意されている。
- [agents/06_review-code.agent.md](agents/06_review-code.agent.md) と配下のレビュー補助エージェント群は、設計整合、規約、セキュリティ、副作用、テスト十分性の観点を分担できる。

## 差分案件として見た推奨運用イメージ

- 既存機能への追加や修正も、新しい case-id を持つ変更案件として扱う。
- 案件として管理する変更要求は差分に絞るが、実装時に参照する文脈は差分に絞らない。周辺の同種機能、同じレイヤ、既存テスト、近い命名や責務分離の書き方を正式入力として扱う。
- 詳細設計書は案件ごとの current な 1 ファイルを更新し続ける。履歴は `source-change-N.md`、handoff、`test-execution-fix-analysis-N.md` など別成果物で持つ。
- 人間は参照コンテキストの完全列挙をしない。変更したいこと、似せたい既存機能、触りたくない領域などのアンカーを自然言語で伝え、エージェントが候補を具体化して確認を返す。
- 02_detailed-design は参照コンテキスト候補をユーザー確認で確定したうえで詳細設計へ反映し、03_implementation はその確定済み基準を読んで実装する。

## 既存定義の修正で解消すべき課題

### P1-1 03_implementation にソース修正の共通作法が不足している

- 事実: [agents/03_implementation.agent.md](agents/03_implementation.agent.md) の入力、成果物、作業フローは `case-manifest.md`、`detailed-design.md`、`implementation-summary.md`、`source-change-N.md` など成果物中心で、修正対象ソースの見つけ方、最小変更の原則、検証コマンド、フォーマッタや lint の扱い、既存変更との共存方針が workspace 共有ルールとして明文化されていない。
- 影響: 実ソース修正の品質がリポジトリ共通ルールではなく、その時のエージェント判断に寄りやすい。ドキュメント更新は揃っても、コード変更の粒度、周辺コードへの合わせ方、検証品質がぶれやすい。
- 対応: 03_implementation が必ず使う実装系スキルを追加し、ソース探索、参照コンテキストの確認、最小変更、関連テスト実行、変更結果の記録を標準化する。`workflow--implementation-source-editing` のような新スキルを追加し、[.github/copilot-instructions.md](copilot-instructions.md) と [agents/03_implementation.agent.md](agents/03_implementation.agent.md) から参照させるのが妥当である。

### P1-2 テスト失敗からの修正で原因分析が任意扱いになっている

- 事実: [agents/03_implementation.agent.md](agents/03_implementation.agent.md) は、テスト実施差し戻し対応で `test-execution-fix-analysis-N.md` を「必要に応じて」作成するとしている。一方で、[docs/templates/test-execution-fix-analysis-template.md](docs/templates/test-execution-fix-analysis-template.md) は原因分析、修正方針、再テスト観点まで持つテンプレートになっている。
- 影響: [agents/05_test-exec.agent.md](agents/05_test-exec.agent.md) から [agents/03_implementation.agent.md](agents/03_implementation.agent.md) へ戻るとき、失敗再現と修正判断の間に一定の分析品質が保証されない。失敗内容をそのままコード修正へ短絡させるリスクがある。
- 対応: `test-execution-to-implementation.md` を正式入力にした場合は、`test-execution-fix-analysis-N.md` を原則必須にするか、後述する原因分析専用エージェントへ委譲する運用に改める。

### P1-3 現行ソースコードの直接修正や緊急修正の入口がない

- 事実: [../outline.md](../outline.md) の状態遷移は 0 から 6 までの標準工程と完了だけで構成されている。また、[agents/03_implementation.agent.md](agents/03_implementation.agent.md) と [skills/workflow--implementation-authority/SKILL.md](skills/workflow--implementation-authority/SKILL.md) は、詳細設計書と実装向け handoff を読む前にコード探索やコード編集を開始しないよう求めている。
- 影響: 「現行ソースの不具合をまず調査したい」「ログや失敗テストから原因を絞りたい」「設計書が未整備でも短期修正したい」といった入口が正規フローに存在しない。
- 対応: 標準運用としては、新しい case を起票して 02 詳細設計を経由する前提を明文化する必要がある。もしそれでは重すぎる運用を許容したいなら、別系統の調査・ホットフィックス用エージェントを追加する必要がある。

### P1-4 05_test-exec の実行権限が広く、ソース修正運用では危険が大きい

- 事実: [agents/05_test-exec.agent.md](agents/05_test-exec.agent.md) には `execute/runInTerminal`、`read/terminalSelection`、`read/terminalLastCommand`、`browser`、`edit/*` が広く付与されている。既存の [problems.md](problems.md) でも権限境界の曖昧さが指摘されている。
- 影響: 実ソース修正までエージェント群に担わせる場合、テスト実施工程が事実上の何でもできる工程になりやすい。誤実行や不要な情報参照のリスクが高い。
- 対応: 実行コマンドの allowlist、対象環境、対象 URL を定義し、不要な terminal 系読み取り権限は削る。必要であれば「テスト実行」と「失敗分析」を別エージェントへ分ける。

### P1-5 06_review-code がレビュー工程なのに編集権限を持っている

- 事実: [agents/06_review-code.agent.md](agents/06_review-code.agent.md) はレビュー工程であるにもかかわらず `edit/createDirectory`、`edit/createFile`、`edit/editFiles`、`edit/rename` を持っている。これも [problems.md](problems.md) で既に指摘されている。
- 影響: レビューと修正の責務が混ざり、コード修正系エージェントを増やしたときの境界がさらに曖昧になる。品質ゲートとしての独立性も落ちる。
- 対応: レビュー工程は review 成果物と handoff の更新だけに限定し、アプリケーションコードやテストコードの編集は禁止事項として明文化する。

### P1-6 03_implementation が工程管理とコード修正を一手に抱えている

- 事実: [agents/03_implementation.agent.md](agents/03_implementation.agent.md) は、成果物更新、handoff 作成、ブラウザ操作、コード編集までまとめて担っている。一方で、テスト仕様作成やレビューには [agents/04-01_test-spec-browser.agent.md](agents/04-01_test-spec-browser.agent.md) から [agents/04-03_test-spec-unit.agent.md](agents/04-03_test-spec-unit.agent.md)、[agents/06-01_review-coding-rules.agent.md](agents/06-01_review-coding-rules.agent.md) から [agents/06-06_review-test-adequacy.agent.md](agents/06-06_review-test-adequacy.agent.md) まで補助エージェントがあるが、実装工程には同種の分割がない。
- 影響: 03 が実質的なスイスアーミーナイフ化し、ソース修正の品質強化や安全制約の追加をしにくい。
- 対応: 少なくとも「原因分析」と「実ソース修正」は補助エージェントへ分離する前提で設計した方が保守しやすい。

### P1-7 差分案件でも参照コンテキストを正式入力として確定する仕組みがない

- 事実: [agents/02_detailed-design.agent.md](agents/02_detailed-design.agent.md) は `detailed-design.md` を作成または更新するが、変更差分に加えて「どの既存実装を基準に合わせるか」を formal に確定する入力や章立てを持っていない。現状のズレ検知は主に [agents/06-01_review-coding-rules.agent.md](agents/06-01_review-coding-rules.agent.md) や [agents/06-03_review-laravel-structure.agent.md](agents/06-03_review-laravel-structure.agent.md) によるレビュー後判定に寄っている。
- 影響: 変更要求を差分だけで管理すると、実装が局所最適化しやすく、完成後に「そこだけ特有の書き方になっている」問題が起きやすい。既存コードベースとの整合が事前ではなく事後にしか分からない。
- 対応: `参照コンテキスト` を正式入力に追加する必要がある。専用成果物として分離するか、`detailed-design.md` の必須章として持たせ、少なくとも同種機能、同じレイヤ、既存テスト、命名と責務分離の参照基準を明記する。

### P1-8 参照コンテキストの指定責務を人間に丸投げすると運用できない

- 事実: 参照コンテキストは必要だが、ファイル、クラス、関数を人間が最初から完全列挙する運用は現実的ではない。変更要求の粒度と参照実装の粒度は一致しないため、起点が粗くても候補抽出はコード探索に依存する。
- 影響: 入力負荷が高すぎると運用が形骸化し、結局は差分要求だけで詳細設計と実装に進んでしまう。逆に細かい指定を強制すると、周辺文脈を広く見るという本来の目的を損なう。
- 対応: 人間は「何を変えたいか」「何に似せたいか」「触りたくない領域はどこか」といった自然言語のアンカーだけを伝え、エージェントが候補を具体化し、ユーザー確認を経て確定する二段構えの運用にするのが妥当である。

### P2-1 テスト不要な修正を閉じる経路が弱い

- 事実: [agents/04_test-spec.agent.md](agents/04_test-spec.agent.md) は、対象テスト種別が 1 件も確定しない場合は停止する。既存の [problems.md](problems.md) でも同じ論点が整理されている。
- 影響: 設定値調整、コメント修正、軽微なテンプレート変更のように、妥当にテスト非対象となるソース修正でフローが詰まりやすい。
- 対応: 03 から 06 へ直接進める条件、または 04 で「テスト不要」を確定して 06 へ送る条件を追加する。

### P2-2 入口文書にソース修正運用の意図が現れていない

- 事実: [copilot-instructions.md](copilot-instructions.md) は、主に workflow 系スキルの入口として構成されており、ソース修正時の共通規律や専用スキルは列挙されていない。
- 影響: カスタムエージェント群として共有したい「実コード修正の標準」が入口文書から読めない。
- 対応: 実装系の新スキルや新エージェントを追加した場合は、入口文書にも役割と参照先を追記する。

## 既存エージェントを修正して対応する場合の推奨方針

1. [agents/03_implementation.agent.md](agents/03_implementation.agent.md) の責務を「工程管理と成果物管理」に寄せ、実ソース修正の作法は新スキルで統一する。
2. 02_detailed-design の冒頭に、差分案件に対する参照コンテキスト候補の抽出とユーザー確認のステップを追加する。候補は同種機能、同じレイヤ、既存テスト、命名と責務分離の観点で整理する。
3. テスト失敗差し戻し時は、`test-execution-fix-analysis-N.md` を原則必須にし、原因分析を省略しない。
4. [agents/05_test-exec.agent.md](agents/05_test-exec.agent.md) はコマンド境界を明確化し、必要なら失敗分析を別エージェントへ切り出す。
5. [agents/06_review-code.agent.md](agents/06_review-code.agent.md) は review 専用に戻し、コード編集権限を剥がす。
6. [agents/04_test-spec.agent.md](agents/04_test-spec.agent.md) と [../outline.md](../outline.md) に、テスト不要なソース修正の完了経路を追加する。
7. [copilot-instructions.md](copilot-instructions.md) に、実装系の新スキルまたは新エージェントを追加したことを反映する。

## 新規追加が妥当なカスタムエージェント

### 案1 02-01_reference-context-curation

- 追加条件: 差分案件を前提にしつつ、詳細設計へ入る前に「従う既存実装の基準」を formal に確定したい場合。
- 主な役割: ユーザーが与えた自然言語のアンカーと既存コードベースをもとに、同種機能、同じレイヤ、既存テスト、近い命名や責務分離の候補を収集し、参照コンテキスト案として返す。
- 想定ツール: `read`, `search`, 必要最小限の `agent`, `todo`
- 想定出力: 参照候補一覧、採用理由、採用しない候補、ユーザー確認が必要な論点
- 効果: 人間がファイルや関数を完全列挙しなくても、02_detailed-design が参照コンテキストを確定してから詳細設計へ進める。

### 案2 03-01_implementation-source-edit

- 追加条件: 03_implementation を親エージェントとして残しつつ、実際のソース探索とコード修正を分離したい場合。
- 主な役割: 詳細設計書、レビュー差し戻し、テスト差し戻しを入力にして、対象ソースの探索、最小変更、ローカル検証、変更結果要約を行う。
- 想定ツール: `read`, `search`, `edit`, 必要最小限の `execute`, `todo`
- 想定出力: 親エージェントへ返す変更ファイル一覧、検証結果、残留リスク
- 効果: 03_implementation から成果物管理と実ソース修正を分離できる。レビューやテストと同様に、実装工程にも役割分割を導入できる。

### 案3 03-02_implementation-failure-analysis

- 追加条件: [agents/05_test-exec.agent.md](agents/05_test-exec.agent.md) からの差し戻しで、失敗再現と原因分析の品質を安定化したい場合。
- 主な役割: `test-result.md`、`test-failures.appendix.md`、`test-execution-to-implementation.md`、必要に応じてログや実行コマンドを読み、`test-execution-fix-analysis-N.md` の草案を作る。
- 想定ツール: `read`, `search`, 必要最小限の `execute`
- 想定出力: 直接原因、根本原因候補、関連実装、修正方針、再テスト観点
- 効果: 失敗内容をそのまま修正へ渡さず、分析を 1 段挟んでから 03_implementation または 03-01_implementation-source-edit へ渡せる。

### 案4 hotfix-implementation

- 追加条件: 詳細設計を起点にする標準フローとは別に、障害対応や短期修正の入口を正式に持ちたい場合。
- 主な役割: issue、ログ、エラー内容、既存ソースコードをもとに、最小限の調査、修正、検証、差分要約を行う。
- 想定ツール: `read`, `search`, `edit`, `execute`, `todo`
- 想定出力: 修正概要、影響範囲、実施コマンド、残留リスク、必要なら後追い設計タスク
- 注意点: これは既存の [../outline.md](../outline.md) を拡張する別系統の運用になるため、標準工程へどこで戻すかまで設計しないと混乱しやすい。通常開発と同列の main agent にするより、緊急対応専用の別入口として扱う方が安全である。

## 推奨結論

- 通常の実装変更までを対象にするなら、新規 main agent を増やす前に [agents/02_detailed-design.agent.md](agents/02_detailed-design.agent.md) と [agents/03_implementation.agent.md](agents/03_implementation.agent.md) と関連スキルを強化する方が先である。
- 具体的には、「参照コンテキストの確定」「実ソース修正の共通作法」「失敗原因分析の標準化」をまず整えるべきである。
- それでも「設計書が薄い既存不具合の修正」や「短期ホットフィックス」まで正式対応したいなら、`03-02_implementation-failure-analysis` と `hotfix-implementation` の追加が妥当である。

## 根拠ファイル

- [agents/03_implementation.agent.md](agents/03_implementation.agent.md)
- [agents/02_detailed-design.agent.md](agents/02_detailed-design.agent.md)
- [agents/04_test-spec.agent.md](agents/04_test-spec.agent.md)
- [agents/05_test-exec.agent.md](agents/05_test-exec.agent.md)
- [agents/06_review-code.agent.md](agents/06_review-code.agent.md)
- [agents/04-01_test-spec-browser.agent.md](agents/04-01_test-spec-browser.agent.md)
- [agents/04-02_test-spec-feature.agent.md](agents/04-02_test-spec-feature.agent.md)
- [agents/04-03_test-spec-unit.agent.md](agents/04-03_test-spec-unit.agent.md)
- [agents/06-01_review-coding-rules.agent.md](agents/06-01_review-coding-rules.agent.md)
- [agents/06-03_review-laravel-structure.agent.md](agents/06-03_review-laravel-structure.agent.md)
- [agents/06-06_review-test-adequacy.agent.md](agents/06-06_review-test-adequacy.agent.md)
- [skills/workflow--implementation-authority/SKILL.md](skills/workflow--implementation-authority/SKILL.md)
- [skills/workflow--common-implement-source-change/SKILL.md](skills/workflow--common-implement-source-change/SKILL.md)
- [docs/templates/test-execution-fix-analysis-template.md](docs/templates/test-execution-fix-analysis-template.md)
- [copilot-instructions.md](copilot-instructions.md)
- [problems.md](problems.md)
- [../outline.md](../outline.md)
