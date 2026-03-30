# ワークフロー調査結果

## 文書情報
- 文書名: ワークフロー調査結果
- 対象: .github 配下の agents / skills / templates / workflow-artifacts
- 作成日: 2026-03-31
- 参照元:
  - .github/copilot-instructions.md
  - .github/agents/*.agent.md
  - .github/skills/*/SKILL.md
  - .github/docs/templates/*.md
  - .github/workflow-artifacts/cases/_case-template/*

## 先に結論

このリポジトリは、以下の 7 工程を基本線にした、文書成果物と handoff を中心に進める開発フローとして設計されています。

1. 基本設計取込
2. 詳細設計
3. 実装
4. テスト仕様作成
5. テスト実施
6. テスト後修正
7. コードレビュー

その上で、コードレビューは終点ではなく、必要に応じて 5. テスト実施へ戻す再テストループを持つ構造です。

ただし、実運用として読むと以下が明確です。

- フローは自動オーケストレーションではなく、実質的に手動進行です。
- 主要な直列工程は定義されていますが、成功時と失敗時の分岐が一部未接続です。
- テスト系のディレクトリ名、handoff の参照ファイル名、レビュー開始条件に矛盾があります。
- 実装成果物や case-manifest 更新責務など、運用に必要な定義が抜けています。

## 調査範囲と読み方

本資料では、内容を以下の 3 区分で記載します。

- 明示定義: ファイル上に明確に書かれているもの
- 実運用推定: 複数ファイルを突き合わせるとそう読むのが自然なもの
- 未定義: 実際に必要だが、定義ファイルに見当たらないもの

## 実際の作業フロー

### 0. 事前準備

明示定義:

- case-id を最初に確認する
- 正式成果物、正式入力、handoff は .github/workflow-artifacts/cases/<case-id>/ 配下に置く
- 同一案件の継続改善では同じ case-id を使う
- case-manifest.md を案件ディレクトリ直下に置き、案件識別情報と現在工程を持つ

実運用推定:

- まずユーザーまたは運用担当が case-id を決める
- 新規案件なら cases/<case-id>/ を _case-template にならって作る
- sources/basic-design/csv/ に元 CSV を置く
- sources/basic-design/source-manifest.md を整備する
- case-manifest.md の 現在工程 を初期化する

未定義:

- case-id 決定の責任者
- case-manifest.md の初期記入例
- source-manifest.md の標準テンプレート

### 1. 基本設計取込

担当エージェント:

- 01_basic-design-import

入力:

- case-manifest.md
- source-manifest.md
- sources/basic-design/csv/*.csv

出力:

- outputs/basic-design/basic-design.md
- handoffs/basic-design-to-detailed-design.md

処理内容:

- CSV 形式の基本設計書を Markdown へ変換する
- 原文忠実性を重視し、意訳や Laravel 実装推測を混ぜない
- 判読困難箇所や解釈分岐は補足や要確認へ逃がす

実運用上の意味:

- この工程は要件や基本設計を Markdown へ正規化し、以後の全工程の基礎資料を作る工程です

### 2. 詳細設計

担当エージェント:

- 02_detailed-design

入力:

- case-manifest.md
- outputs/basic-design/basic-design.md
- handoffs/basic-design-to-detailed-design.md

出力:

- outputs/detailed-design/detailed-design.md
- handoffs/detailed-design-to-implementation.md

処理内容:

- 基本設計の意図を保ちつつ、Laravel の責務分離を意識した詳細設計へ展開する
- 変更対象と非変更対象を分ける
- 条件分岐、例外、更新仕様、バリデーションを構造化する

実運用上の意味:

- この工程で、実装の正式判断基準になる文書を作る

### 3. 実装

担当エージェント:

- 03_implementation

入力:

- case-manifest.md
- outputs/basic-design/basic-design.md
- outputs/detailed-design/detailed-design.md
- handoffs/detailed-design-to-implementation.md

出力:

- outputs/implementation/implementation-summary.md
- outputs/implementation/source-change-01.md
- handoffs/implementation-to-test-specification.md

処理内容:

- 詳細設計を主基準に実装する
- 基本設計は意図確認用の参考資料として扱う
- 設計差分が出た場合は変更一覧に理由を残す

実運用上の意味:

- コード変更そのものだけでなく、後工程が読める変更記録を残す工程です

### 4. テスト仕様作成

担当エージェント:

- 04_test-spec

入力:

- outputs/basic-design/basic-design.md
- outputs/detailed-design/detailed-design.md
- outputs/implementation/implementation-summary.md
- outputs/implementation/source-change-01.md
- handoffs/implementation-to-test-specification.md

出力:

- outputs/testing/test-spec-browser.md
- outputs/testing/test-spec-feature.md
- outputs/testing/test-spec-unit.md
- handoffs/test-specification-to-test-execution.md

処理内容:

- 観点をブラウザ、Feature、Unit に分離してテスト仕様を作る
- handoff から実装差分や回帰リスクを取り込む

実運用上の意味:

- テストを先に仕様化して、以後の実施とレビューの基準を固定する工程です

### 5. テスト実施

担当エージェント:

- 05_test-exec

入力:

- outputs/testing/test-spec-browser.md
- outputs/testing/test-spec-feature.md
- outputs/testing/test-spec-unit.md
- handoffs/test-specification-to-test-execution.md
- handoffs/code-review-to-test-execution.md が存在する場合はそれも入力

出力:

- outputs/testing/test-result.md
- outputs/testing/test-failures.appendix.md
- handoffs/test-execution-to-post-test-fix.md

処理内容:

- テスト仕様に基づいてブラウザ、Feature、Unit を実施する
- 初回テストとレビュー後再テストを test-result.md の 区分 で分ける想定になっている

実運用上の意味:

- 単なる実行ではなく、再現可能な失敗条件を後続工程へ渡すための証跡化工程です

### 6. テスト後修正

担当エージェント:

- 06_post-test-fix

入力:

- case-manifest.md
- outputs/testing/test-result.md
- outputs/testing/test-failures.appendix.md
- handoffs/test-execution-to-post-test-fix.md

出力:

- outputs/implementation/source-change-02.md
- outputs/implementation/post-test-fix-analysis.md
- handoffs/post-test-fix-to-code-review.md

処理内容:

- テスト失敗の再現条件を確認した上で、根本原因に対処する
- 変更点、原因分析、残留リスク、再テスト推奨箇所を記録する

実運用上の意味:

- テスト失敗を受けた修正専用工程です

### 7. コードレビュー

担当エージェント:

- 07_review-code
- 必要に応じて 07-01 から 07-06 の補助レビューエージェントを並列実行

入力:

- case-manifest.md
- outputs/basic-design/basic-design.md
- outputs/detailed-design/detailed-design.md
- outputs/implementation/source-change-02.md
- outputs/implementation/post-test-fix-analysis.md
- handoffs/post-test-fix-to-code-review.md

出力:

- outputs/review/review-result.md
- outputs/review/review-findings.appendix.md
- handoffs/code-review-to-test-execution.md

処理内容:

- 設計整合
- コーディング規約
- Laravel の責務配置
- セキュリティ
- 副作用、回帰
- テスト十分性

実運用上の意味:

- レビュー結果を判定書として出しつつ、必要な再テスト観点を handoff へ返す品質ゲートです

### 8. 再テストループ

明示定義:

- code-review-to-test-execution.md はレビュー後再テストの正式入力
- review-result.md は判定結果であり、次工程入力とは混同しない
- 05_test-exec は code-review-to-test-execution.md が存在する場合を条件付き入力としている

実運用推定:

- 07_review-code で再テスト観点が出たら 05_test-exec を再度実行する
- 再テスト結果を test-result.md に追記または上書きする想定だが、方式は未定義

## 条件分岐

### A. 明示定義されている分岐

#### A-1. case-id の有無

- 既存 case-id がある: 継続案件として扱う
- 既存 case-id がない: 新規案件として cases/<case-id>/ を用意する

#### A-2. 入力ファイルが揃っているか

- 揃っている: 工程を進める
- 足りない: 不足を明記する

補足:

- 不足情報は空欄にせず、要確認として扱う方針は明示されています
- ただし工程停止するのか、一部保留で進めるのかまでは定義されていません

#### A-3. テスト観点の分離

- 画面、操作、表示、遷移中心: ブラウザテスト
- HTTP 層、認証、認可、Request、Controller 連携: Feature テスト
- 純粋ロジック、計算、例外、分岐: Unit テスト

#### A-4. レビュー後 handoff の存在有無

- code-review-to-test-execution.md がある: レビュー後再テスト
- ない: 初回テスト

#### A-5. コードレビューの観点別サブエージェント実行

- 07_review-code は必要に応じて 07-01 から 07-06 を使う
- 実行方針として並列レビューが書かれている

### B. 実運用上そう読むのが自然な分岐

#### B-1. テスト失敗あり / なし

- 失敗あり: 06_post-test-fix に進む
- 失敗なし: 本来はレビューへ直接進みたい

問題点:

- 定義ファイル上、05_test-exec の handoff 先は 06_post-test-fix のみです
- 失敗なし時の正式な次工程が agent 定義上にありません

#### B-2. レビュー指摘あり / なし

- 指摘あり: 再テスト観点を返して 05_test-exec へ戻す
- 指摘なし: 承認して終了したい

問題点:

- 完了条件が明文化されていません
- 何をもって案件終了とするか、case-manifest をどう更新するかが未定義です

#### B-3. 詳細設計が存在する / しない

- 存在する: 実装の主基準にする
- 存在しない: どうするか未定義

補足:

- ルールは 詳細設計が存在する工程では詳細設計を主基準 としており、存在しない場合の代替フローは書かれていません

#### B-4. どのテスト種別を作るか

- 3 種すべて毎回作るのか
- 変更内容に応じて一部だけ作るのか

これはテンプレート上も agent 定義上も未定義です。

### C. 未定義だが実運用で必須になる分岐

#### C-1. 重大レビュー指摘が出たとき、修正へ戻るのか再テストへ戻るのか

- 現在の正式 handoff は 07_review-code から 05_test-exec へのみ定義されています
- しかし設計逸脱や実装修正が必要な指摘は、05_test-exec では解決できません

#### C-2. 再テストでも失敗した場合の再ループ経路

- 05 -> 06 -> 07 -> 05 のループは読めます
- 2 周目以降の source-change-03.md 以降の扱いは未定義です

#### C-3. 複数機能を同一 case-id で並行するかどうか

- case-id の再利用原則はある
- しかし複数テーマを同時進行した際の成果物分割ルールはありません

## 実務としての運用イメージ

実際に案件を進めるなら、次のように読むのが最も自然です。

1. case-id を決める
2. cases/<case-id>/ を雛形通りに作る
3. source-manifest.md と CSV を配置する
4. 01_basic-design-import を実行する
5. 02_detailed-design を実行する
6. 03_implementation を実行する
7. 04_test-spec を実行する
8. 05_test-exec を実行する
9. 失敗があれば 06_post-test-fix へ進む
10. 07_review-code を実行する
11. レビューで再テスト観点が返れば 05_test-exec を再実行する
12. 承認と見なせる状態になったら、case-manifest を完了へ更新する

重要な実務ポイント:

- 各 agent の handoffs はすべて send: false なので、自動的に次工程へ飛ぶのではなく、運用者が都度起動する前提です
- したがって、この仕組みは ワークフローエンジン というより 工程定義付きの半手動運用フレーム と捉えるのが正確です

## 見つかった課題

### 重大

#### 1. テスト成果物ディレクトリ名が衝突している

- skills/workflow__common_artifact-location/SKILL.md では outputs/test/
- agents と _case-template では outputs/testing/

影響:

- 成果物配置先が文書ごとに食い違う
- 次工程が入力を見失う危険がある

#### 2. 04_test-spec の handoff prompt が存在しないファイルを参照している

- handoff prompt は outputs/testing/test-specification.md を前提にしている
- 実際の成果物定義は test-spec-browser.md、test-spec-feature.md、test-spec-unit.md の 3 ファイル

影響:

- 次工程への受け渡しメッセージが実態と不一致
- 運用者が誤ったファイルを探す

#### 3. テスト成功時の正式な遷移先が欠落している

- 05_test-exec の handoff は 06_post-test-fix のみ
- 06_post-test-fix は失敗原因を特定して修正する工程

影響:

- 全件成功でも定義上は修正工程へしか進めない
- 正常系フローが agent 定義上で閉じていない

#### 4. 07_review-code が初回実装のままでは入力を満たせない

- 07_review-code の入力は source-change-02.md と post-test-fix-analysis.md 前提
- つまり 06_post-test-fix を経ないとレビュー入力が揃わない

影響:

- テスト失敗が一度も無い案件をレビューできない構造になっている
- 実務上あり得る正常パスが欠落している

### 中程度

#### 5. case-manifest の更新責務が未定義

- 現在工程は最新状態へ更新する と case-manifest に書かれている
- しかし、どの工程がいつ更新するかは書かれていない

影響:

- 実運用で進捗管理が人依存になる

#### 6. 実装成果物のテンプレートがない

- implementation-summary.md
- source-change-01.md
- source-change-02.md
- post-test-fix-analysis.md

影響:

- 記載粒度が人ごとにぶれる
- レビューやテスト仕様作成で必要情報が欠けやすい

#### 7. case-manifest と source-manifest の標準化が弱い

- case-manifest は _case-template に実体があるが docs/templates 配下にない
- source-manifest はケース雛形に存在するが、テンプレート運用ルールが弱い

影響:

- 新規案件時の初期化品質が安定しない

#### 8. レビュー指摘が修正を要する場合の戻り先が未定義

- 07_review-code からは 05_test-exec への再テスト handoff しかない
- 実装修正が必要な指摘に対する正式な戻り経路がない

影響:

- 設計逸脱、セキュリティ欠陥、責務違反のような問題をどの工程で直すか曖昧

### 軽微

#### 9. テンプレート参照パス記法が揺れている

- 絶対パス変数形式と相対リンク形式が混在

#### 10. .github/instructions ディレクトリが空

- 構造としては存在するが、全体運用に追加命令は置かれていない

#### 11. 出力先指定と標準配置ルールが衝突している

- 標準ルールでは正式成果物は cases/<case-id>/ 配下に置く
- 今回の依頼では .github/result.md を指定されている

影響:

- このファイルは調査報告としては有用だが、リポジトリが定義する正式成果物配置ルールには一致しない

## 未定義と推測できる部分

### 未定義

- 工程完了条件
- 承認後の終了処理
- テスト成功時のレビュー遷移
- レビュー指摘時の修正遷移
- 再テスト結果の上書きか追記か
- source-change-03.md 以降の採番
- 複数機能並行時のディレクトリ分割
- どの変更でブラウザ / Feature / Unit を省略可能か
- 入力不足時に 停止 か 継続か の基準

### 推測でしか補えない部分

- 実際には 05_test-exec 成功時に 07_review-code へ直接進めたいはず
- 実際には 07_review-code の重大指摘時に 03_implementation か 06_post-test-fix へ戻したいはず
- 実際には case-manifest が工程の都度更新される想定のはず
- 実際には test-result.md は同一ファイルに区分を分けて追記する想定の可能性が高い

## 改善優先順位

### 優先度 1

- outputs/test と outputs/testing を統一する
- 04_test-spec の handoff prompt を 3 つの test-spec ファイルに合わせて修正する
- 05_test-exec 成功時の次工程を正式定義する
- 07_review-code が source-change-01.md ベースでもレビュー可能になるよう入力定義を見直す

### 優先度 2

- 実装成果物テンプレートを追加する
- case-manifest 更新責務を各 agent に明記する
- レビュー指摘から修正工程へ戻る正式ルートを追加する
- 再テスト時の test-result.md 更新ルールを定義する

### 優先度 3

- case-manifest と source-manifest を docs/templates 配下の正式テンプレートへ昇格する
- パス表記と命名規則を統一する
- 複数テーマ並行時の成果物分割ルールを追加する

## 最終整理

このリポジトリの設計思想自体は一貫しています。特に以下は明確です。

- 工程ごとに責務を分離する
- 文書成果物を正式入力として次工程へ渡す
- 詳細設計を実装の主基準にする
- レビューを品質ゲートとして再テストへ返す

一方で、実際の作業フローとして見ると、現在の定義は 失敗時の修正ループ は比較的見える一方、成功時の直進ルート と レビュー指摘後の修正ルート が弱く、そこで運用者判断に依存します。

したがって、現状は 厳密に自動進行できる完成済みワークフロー ではなく、強い工程意図を持つ半手動運用フレーム と評価するのが妥当です。

## 補足

今回の調査では、runSubagent を 8 系統に分けて並列探索し、agents、skills、templates、case template、条件分岐、矛盾、実運用再構成を突き合わせて本資料を作成しました。
