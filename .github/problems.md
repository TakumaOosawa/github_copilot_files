# .github 配下 GitHub Copilot 設定ファイルの課題一覧

## 調査方法

- 対象は [copilot-instructions.md](copilot-instructions.md)、[agents](agents)、[skills](skills)、[docs/templates](docs/templates)、[docs/templates/_case-template](docs/templates/_case-template) とした。
- 10 本の runSubagent で観点別に調査し、優先度が高い候補は read_file と list_dir で一次証跡を再確認した。
- 優先度は、P1 を誤作動・誤誘導・工程停止、P2 を安全性と運用信頼性の低下、P3 を保守性と発見性の低下として整理した。

## P1

### P1-1 基本設計成果物の正規配置が分裂している

- 対象: [skills/workflow--common-artifact-location/SKILL.md](skills/workflow--common-artifact-location/SKILL.md)、[agents/00_case-init.agent.md](agents/00_case-init.agent.md)、[agents/01_basic-design-import.agent.md](agents/01_basic-design-import.agent.md)、[agents/02_detailed-design.agent.md](agents/02_detailed-design.agent.md)、[docs/templates/case-manifest-template.md](docs/templates/case-manifest-template.md)、[docs/templates/_case-template](docs/templates/_case-template)
- 事実: 共通ルールと 00 は outputs/basic-design を正規配置として扱っている一方、01 と 02 は outputs/basic-design/markdown 配下を正式入力として扱い、case-manifest テンプレートは outputs/basic-design/basic-design.md を追跡対象にしている。さらに _case-template の実体には sources/basic-design/markdown ディレクトリがある。
- 影響: 初期化、案件管理、後続工程が別々の正規配置を前提にするため、新規案件の立ち上げと再開で迷子になりやすい。テンプレート更新時の修正漏れも起きやすい。
- 推奨対応: 基本設計 Markdown の正規配置を 1 つに固定し、共通 skill、00、case-manifest テンプレート、_case-template、01、02 を同時にそろえる。

### P1-2 00_case-init の handoff prompt が self-contained になっていない

- 対象: [agents/00_case-init.agent.md](agents/00_case-init.agent.md)
- 事実: 01 への handoff prompt だけが source-manifest.md と handoff ファイルを cases/<case-id> 配下の相対断片で記述しており、他の handoff prompt のような完全パス列挙になっていない。
- 影響: handoff prompt だけを読んだ次工程が正式入力を機械的に特定しにくい。prompt の self-contained 性が崩れ、運用ルールの一貫性も落ちる。
- 推奨対応: prompt 内で参照する全ファイルを ${workspaceFolder}/.github/workflow-artifacts/cases/<case-id>/... の完全パスで列挙する。

### P1-3 04_test-spec から 03_implementation への handoff prompt が 03 の正式入力を満たしていない

- 対象: [agents/04_test-spec.agent.md](agents/04_test-spec.agent.md)、[agents/03_implementation.agent.md](agents/03_implementation.agent.md)
- 事実: 04 の 03 向け handoff prompt には、03 が正式入力として定義している基本設計 Markdown 群と detailed-design-to-implementation.md が含まれていない。
- 影響: handoff prompt を信頼して 03 を再開すると、callee 側が前提にしている入力契約とずれる。差し戻しループで必要情報の読み落としが起きやすい。
- 推奨対応: 04 の 03 向け prompt を、03 の正式入力を最低限包含する内容へ修正する。あわせて handoff prompt は callee の正式入力を包含するという共通ルールを明文化する。

### P1-4 review-result テンプレートの次アクションが 06_review-code の正式分岐と一致していない

- 対象: [docs/templates/review-result-template.md](docs/templates/review-result-template.md)、[agents/06_review-code.agent.md](agents/06_review-code.agent.md)
- 事実: review-result テンプレートの次アクションは 完了、実装修正、テスト仕様更新、再テスト の 4 通りしかないが、06 は code-review-to-detailed-design.md を作成して 02 へ戻す正式分岐を持っている。
- 影響: レビュー結果をテンプレートどおりに埋めると、詳細設計への差し戻しを正規の語彙で表現できない。レビュー結果と handoff の整合が崩れ、誤ルーティングを招きやすい。
- 推奨対応: 次アクションに 詳細設計更新 もしくは同義の選択肢を追加し、判定ルールも 06 の分岐条件に合わせる。

### P1-5 すべてのテスト種別が非対象になった場合の完了経路がない

- 対象: [skills/workflow--common-implement-summary/SKILL.md](skills/workflow--common-implement-summary/SKILL.md)、[agents/03_implementation.agent.md](agents/03_implementation.agent.md)、[agents/04_test-spec.agent.md](agents/04_test-spec.agent.md)
- 事実: implementation-summary では 3 種別すべてに対象、非対象、要確認の判定記載を要求している一方、04 は対象テスト種別が 1 件も確定しないと停止する。03 からの通常遷移先は 04 であり、テスト不要として閉じる分岐がない。
- 影響: 文言修正や設定変更のように 3 種別すべてが妥当に非対象な変更で工程が詰まる。不要なテスト種別を捏造する圧力も生まれる。
- 推奨対応: テスト不要の正式分岐を追加するか、0 件対象を許容してレビューへ進める規約を用意する。

## P2

### P2-1 05_test-exec の権限が広すぎ、環境境界も定義されていない

- 対象: [agents/05_test-exec.agent.md](agents/05_test-exec.agent.md)
- 事実: runInTerminal、terminalSelection、terminalLastCommand、browser、web/fetch が広く許可されている一方、許可コマンド、対象環境、対象 URL の境界が書かれていない。
- 影響: テスト以外の環境での誤実行、端末選択内容や直前コマンドからの機密露出、不要なブラウザ操作のリスクがある。
- 推奨対応: 実行コマンドの allowlist、対象環境の明示確認、対象 URL の制限を追加し、terminalSelection と terminalLastCommand は削る。

### P2-2 06_review-code がレビュー工程なのに編集権限を持ち、ソース編集禁止も明記していない

- 対象: [agents/06_review-code.agent.md](agents/06_review-code.agent.md)
- 事実: review 親エージェントに createDirectory、createFile、editFiles、rename が付与されているが、禁止事項にはソースコード、テストコード、設計書本文を編集しない制約がない。
- 影響: レビューと修正の責務が混線し、review-result と実ソースの独立性が崩れる。監査と再レビューの観点でも不利。
- 推奨対応: 書き込み先を outputs/review と handoffs に限定するか、少なくともレビュー工程でのソース編集禁止を明文化する。

### P2-3 06 系の補助エージェントで責務が重複している

- 対象: [agents/06-01_review-coding-rules.agent.md](agents/06-01_review-coding-rules.agent.md)、[agents/06-03_review-laravel-structure.agent.md](agents/06-03_review-laravel-structure.agent.md)、[agents/06-05_review-security.agent.md](agents/06-05_review-security.agent.md)、[agents/06-02_review-design-alignment.agent.md](agents/06-02_review-design-alignment.agent.md)
- 事実: 06-01、06-03、06-05 がいずれも Laravel の責務配置を確認対象に含め、06-03 は設計差分まで扱っていて 06-02 とも境界がぶつかっている。
- 影響: 同一論点の重複指摘、逆に相手担当待ちによる見落とし、親エージェント側の重複排除コストが発生する。
- 推奨対応: 06-03 を Laravel 構造の主担当に固定し、06-01 は規約と可読性、06-05 はセキュリティ結果に限定する。補助エージェントの出力契約も共通化する。

### P2-4 共通出力ルールの 1 工程 1 正式ファイル という原則が現行 workflow と衝突している

- 対象: [skills/workflow--common-output-format/SKILL.md](skills/workflow--common-output-format/SKILL.md)、[agents/03_implementation.agent.md](agents/03_implementation.agent.md)、[agents/04_test-spec.agent.md](agents/04_test-spec.agent.md)
- 事実: common-output-format は 1 工程 1 正式ファイルを原則としているが、03 は implementation-summary と source-change-N を併用し、04 は test-spec-browser、test-spec-feature、test-spec-unit を正式成果物として持っている。
- 影響: 共通ルールを正本として読むと、現行の複数成果物工程が例外運用になってしまう。新規 agent 追加時に誤ったルール解釈を招く。
- 推奨対応: common-output-format を書式と記述ルールに限定するか、複数正式成果物を持つ工程の例外を明記する。

### P2-5 00_case-init の既存案件再開方針と実際の handoff 制約が一致していない

- 対象: [agents/00_case-init.agent.md](agents/00_case-init.agent.md)
- 事実: 本文では既存案件なら現在工程を見て適切なエージェントへの引継ぎを案内するとしている一方、実際の frontmatter handoff は 01_basic-design-import だけで、本文でも handoffs に記載されたエージェント群への引継ぎのみ可能としている。
- 影響: 入口エージェントとしての説明と実際に取れる遷移がずれ、既存案件再開時に誤案内が起きやすい。
- 推奨対応: 00 を新規案件専用と明記するか、現在工程に応じた handoff を追加する。

## P3

### P3-1 見出し構文と見出し名の小さな崩れが残っている

- 対象: [agents/02_detailed-design.agent.md](agents/02_detailed-design.agent.md)、[agents/03_implementation.agent.md](agents/03_implementation.agent.md)、[agents/04_test-spec.agent.md](agents/04_test-spec.agent.md)
- 事実: 02 の スキル見出しだけが #本 になっており Markdown 見出しとして崩れている。加えて 03 と 04 の成果物見出しだけが 本エージェントの成果物ファイル で、他の main agent と一致していない。
- 影響: VS Code のアウトライン、差分レビュー、将来の lint 導入時にノイズになる。
- 推奨対応: main agents 00 から 06 の見出しラベルを完全一致で正規化する。

### P3-2 入口文書の skill 体系説明が実体と少しずれている

- 対象: [copilot-instructions.md](copilot-instructions.md)、[agents/00_case-init.agent.md](agents/00_case-init.agent.md)、[skills/common--fetch-current-datetime/SKILL.md](skills/common--fetch-current-datetime/SKILL.md)
- 事実: 入口文書は 詳細ルールは workflow--* スキルに分割して管理すると説明しているが、実際には common--fetch-current-datetime も main agent から使われている。
- 影響: 新規 contributor が common 系 skill の存在を見落としやすい。命名規約の理解もぶれやすい。
- 推奨対応: .github/skills 配下の skill に分割して管理する、と表現を緩めるか、workflow 系と common 系の二系統を明示する。

### P3-3 外部サイト依存の日時取得 skill が不要な外部依存を増やしている

- 対象: [skills/common--fetch-current-datetime/SKILL.md](skills/common--fetch-current-datetime/SKILL.md)、[agents/00_case-init.agent.md](agents/00_case-init.agent.md)
- 事実: 現在日時取得のために毎回 worldtimeserver.com へ fetch し、キャッシュ回避の unique_id まで要求している。
- 影響: 外部サービス停止時の失敗点になり、日時記録という軽い用途のために不要な外部通信面を増やしている。
- 推奨対応: セッション文脈またはローカル時刻取得へ寄せる。外部参照が必要な場合も first-party か明示的な allowlist に限定する。

## 優先度を下げたが把握済みの論点

- 表記ゆれは別途 [valiation-in-spelling.md](valiation-in-spelling.md) に 8 件まとまっており、今回の problems.md では動作影響が大きい問題を優先した。
- code-review-complete.md は from-to 形式の handoff 命名規約から外れているが、現時点では命名ルールの一貫性問題として扱い、上位課題には入れていない。

## 除外した誤検知

- [.github/instructions](instructions) が空なのは、現行運用では workflow ルールを skill へ集約しているため、単独では問題としなかった。
- agent と skill の frontmatter 区切り、および主要な参照名の整合は概ね成立していた。
- docs/templates 配下で参照されている主要テンプレートファイルは存在していた。
