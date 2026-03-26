**1. 最新 Copilot を踏まえた設計方針**

今回の仕様は、GitHub Copilot の最新 overview にある次の考え方に合わせます。

1. エージェントを工程ごとに分ける
2. セッションを持ちながら対話で成果物をブラッシュアップする
3. 工程間は handoff で明示的に切り替える
4. Custom agents、Custom instructions、Agent skills を組み合わせる
5. ブラウザテストは統合ブラウザを前提に扱う
6. コードレビューは複数観点の agent を使って分解する
7. 永続成果物は案件単位のディレクトリで分離する
8. コードレビューの handoff 先は test-execution とし、レビュー結果は再テストの正式入力に含める

この前提だと、**作業途中の状態はセッション、正式成果物はファイル、案件の分離はディレクトリ、品質確認のループは review から test-execution へ戻す**という分け方が最も分かりやすいです。

---

**2. 全体構成**

.github の中は、役割ごとに次の5系統に分けます。

```text
.github/
  copilot-instructions.md
  agents/
    basic-design-md.agent.md
    detailed-design.agent.md
    implementation.agent.md
    test-specification.agent.md
    test-execution.agent.md
    post-test-fix.agent.md
    code-review.agent.md
    review-design-alignment.agent.md
    review-coding-rules.agent.md
    review-security.agent.md
    review-regression.agent.md
    review-test-adequacy.agent.md
    review-laravel-structure.agent.md
  instructions/
    workflow/
      handoff-format.instructions.md
      markdown-output.instructions.md
      artifact-location.instructions.md
    design/
      basic-design-fidelity.instructions.md
      detailed-design-structure.instructions.md
    implementation/
      implementation-authority.instructions.md
    testing/
      testing-viewpoints.instructions.md
      browser-testing.instructions.md
      phpunit-testing.instructions.md
    review/
      review-scope.instructions.md
  skills/
    browser-test-execution/
      SKILL.md
    phpunit-test-execution/
      SKILL.md
    design-template-guidance/
      SKILL.md
  docs/
    templates/
      basic-design-md-template.md
      detailed-design-template.md
      test-specification-template.md
      test-result-template.md
      review-result-template.md
      handoff-template.md
  workflow-artifacts/
    cases/
      <case-id>/
        case-manifest.md
        sources/
          basic-design/
            source-manifest.md
            csv/
        outputs/
          basic-design/
          detailed-design/
          implementation/
          testing/
          review/
        handoffs/
```

重要なのは、`workflow-artifacts/` 直下にすぐ `sources/` や `outputs/` を置かず、**必ず `cases/<case-id>/` を挟む**ことです。

---

**3. 何をどこに置くか**

**3.1 エージェント定義**

置き場所:

```text
.github/agents/
```

内容:
1. 7体の主エージェント
2. コードレビュー用の補助エージェント

補助エージェントは agents 直下に置き、`user-invocable: false` にする方針でよいです。

---

**3.2 共通指示**

置き場所:

```text
.github/copilot-instructions.md
```

内容:
1. 全エージェント共通の原則
2. 責務分離
3. 対話で成果物を磨くこと
4. 詳細設計優先
5. handoff を必ず残すこと
6. 成果物は必ず `workflow-artifacts/cases/<case-id>/` 配下に置くこと

---

**3.3 補助 instruction**

置き場所:

```text
.github/instructions/
```

内容:
1. 工程別ルール
2. レビュー観点
3. テスト観点
4. Markdown 出力の統一ルール
5. handoff の書き方
6. 案件ディレクトリの選び方と成果物配置ルール

---

**3.4 skills**

置き場所:

```text
.github/skills/
```

内容:
1. 統合ブラウザテストの実行ノウハウ
2. PHPUnit 実行ノウハウ
3. テンプレートへの落とし込み手順

---

**3.5 テンプレート**

置き場所:

```text
.github/docs/templates/
```

内容:
1. 基本設計 md テンプレート
2. 詳細設計テンプレート
3. テスト仕様書テンプレート
4. テスト結果テンプレート
5. レビュー結果テンプレート
6. handoff テンプレート

---

**3.6 成果物**

置き場所:

```text
.github/workflow-artifacts/cases/<case-id>/
```

内容:
1. 外部から入ってくるソース
2. 各工程の正式成果物
3. 次工程に渡す handoff メモ
4. 案件の識別情報を持つ manifest

ここでのポイントは、**成果物は repository 共通領域に直接積まず、必ず案件ルートの中に閉じ込める**ことです。

---

**4. `.working.md` と `.final.md` の再整理**

前回案では 2 つに分けていましたが、今回はやめます。

**再整理後のルール**
1. 作業途中の調整はチャットセッションで行う
2. ファイルとして残すのは正式成果物だけ
3. 迷いや未確定事項は成果物本文か handoff に明示する
4. よって、各工程の成果物は原則 1 ファイルにする
5. ただし成果物の所属先は必ず案件ディレクトリごとに分ける

つまり、例えば基本設計はこう変えます。

変更前:
```text
.github/workflow-artifacts/outputs/basic-design/basic-design.working.md
.github/workflow-artifacts/outputs/basic-design/basic-design.final.md
```

変更後:
```text
.github/workflow-artifacts/cases/<case-id>/outputs/basic-design/basic-design.md
```

このほうが運用しやすいです。

---

**5. 成果物の分類ルール**

成果物ファイルは次の3種類だけにします。

**5.1 source**
人間側が最初に与える元資料

**5.2 output**
各工程の正式成果物

**5.3 handoff**
次工程向けの引継ぎメモ  
これは正式入力に含める

**5.4 case directory**
案件ごとの永続領域

このディレクトリは `workflow-artifacts/cases/<case-id>/` とし、その中に `sources`、`outputs`、`handoffs` を閉じ込めます。

**5.5 case-id の命名ルール**

推奨ルール:
1. 小文字英数字とハイフンで表現する
2. チケット番号や業務名が分かる短い識別子を含める
3. 必要なら日付プレフィックスを付ける

例:
```text
feature-user-import
bugfix-login-lock
crm-142-order-export
20260324-invoice-download
```

同じ案件を継続して磨く間は同じ `case-id` を使い、別案件なら新しい `case-id` を切ります。

**5.6 case-manifest.md**

各案件ディレクトリ直下に次を置きます。

```text
.github/workflow-artifacts/cases/<case-id>/case-manifest.md
```

ここには次を書く想定です。

1. 案件名
2. case-id
3. 関連チケットや依頼番号
4. 対象システムや対象機能
5. 開始日
6. 現在の工程
7. 備考

これを置いておくと、エージェントがどの案件ディレクトリを使うべきか判断しやすくなります。

---

**6. 基本設計md化エージェント仕様**

**6.1 役割**
CSV 化された基本設計資料を、原文忠実に Markdown 化する。

**6.2 対象エージェント**
```text
.github/agents/basic-design-md.agent.md
```

**6.3 補助 instruction**
```text
.github/instructions/design/basic-design-fidelity.instructions.md
.github/instructions/workflow/artifact-location.instructions.md
```

**6.4 テンプレート**
```text
.github/docs/templates/basic-design-md-template.md
```

**6.5 正式入力**
```text
.github/workflow-artifacts/cases/<case-id>/case-manifest.md
.github/workflow-artifacts/cases/<case-id>/sources/basic-design/source-manifest.md
.github/workflow-artifacts/cases/<case-id>/sources/basic-design/csv/*.csv
```

**6.6 入力ファイルに何を書くか**

`case-manifest.md`
1. 案件識別情報
2. 対象機能
3. 現在工程
4. 関連チケット
5. 備考

`source-manifest.md`
1. 元の設計書名
2. CSV 一覧
3. 各 CSV の意味
4. シート相当の順序
5. 例外シートや読み方の注意点

`csv/*.csv`
1. 基本設計書を CSV 化した実データ
2. AI はここを原文ソースとして読む

**6.7 正式出力**
```text
.github/workflow-artifacts/cases/<case-id>/outputs/basic-design/basic-design.md
.github/workflow-artifacts/cases/<case-id>/handoffs/basic-design-to-detailed-design.md
```

**6.8 出力ファイルに何を書くか**

`basic-design.md`
1. 基本文書情報
2. 機能概要
3. 業務ルール
4. 画面・帳票・IF 情報
5. 項目定義
6. 制約事項
7. 原文補足が必要な箇所
8. 要確認事項

重要:
本文は原則転記
補足や懸念は別章

`basic-design-to-detailed-design.md`
1. 参照した CSV 一覧
2. 重要な読替えなしの前提
3. 曖昧点
4. 詳細設計で重点的に構造化すべき箇所
5. 次工程への注意

---

**7. 詳細設計エージェント仕様**

**7.1 役割**
基本設計をもとに、処理経路が追いやすい詳細設計を作る。

**7.2 対象エージェント**
```text
.github/agents/detailed-design.agent.md
```

**7.3 補助 instruction**
```text
.github/instructions/design/detailed-design-structure.instructions.md
.github/instructions/workflow/artifact-location.instructions.md
```

**7.4 テンプレート**
```text
.github/docs/templates/detailed-design-template.md
```

**7.5 正式入力**
```text
.github/workflow-artifacts/cases/<case-id>/outputs/basic-design/basic-design.md
.github/workflow-artifacts/cases/<case-id>/handoffs/basic-design-to-detailed-design.md
```

**7.6 入力ファイルに何を書くものか**

`basic-design.md`
基本設計の正式版。詳細設計の元資料。

`basic-design-to-detailed-design.md`
詳細設計時に見落としやすい注意点、曖昧点、重点整理ポイント。

**7.7 正式出力**
```text
.github/workflow-artifacts/cases/<case-id>/outputs/detailed-design/detailed-design.md
.github/workflow-artifacts/cases/<case-id>/handoffs/detailed-design-to-implementation.md
```

**7.8 出力ファイルに何を書くか**

`detailed-design.md`
1. 処理概要
2. 変更対象一覧
3. 非変更対象一覧
4. Route
5. Middleware
6. Controller
7. Request
8. Service
9. Repository
10. Model
11. View
12. 処理フロー
13. 条件分岐
14. バリデーション
15. 例外処理
16. 更新仕様
17. ログ・監査対象
18. 実装上の注意点

`detailed-design-to-implementation.md`
1. 実装優先ポイント
2. 設計上の曖昧点
3. 設計上の制約
4. 変更対象ファイル候補
5. 実装時に独断禁止の箇所

---

**8. 実装エージェント仕様**

**8.1 役割**
詳細設計を優先してコード修正を行う。
基本設計は意図確認用の参考資料。

**8.2 対象エージェント**
```text
.github/agents/implementation.agent.md
```

**8.3 補助 instruction**
```text
.github/instructions/implementation/implementation-authority.instructions.md
.github/instructions/workflow/artifact-location.instructions.md
```

**8.4 正式入力**
```text
.github/workflow-artifacts/cases/<case-id>/outputs/basic-design/basic-design.md
.github/workflow-artifacts/cases/<case-id>/outputs/detailed-design/detailed-design.md
.github/workflow-artifacts/cases/<case-id>/handoffs/detailed-design-to-implementation.md
```

**8.5 入力ファイルに何を書くものか**

`basic-design.md`
元の意図確認用

`detailed-design.md`
実装の主基準

`detailed-design-to-implementation.md`
実装時の注意、保留判断、要確認ポイント

**8.6 正式出力**
```text
.github/workflow-artifacts/cases/<case-id>/outputs/implementation/implementation-summary.md
.github/workflow-artifacts/cases/<case-id>/outputs/implementation/source-change-01.md
.github/workflow-artifacts/cases/<case-id>/handoffs/implementation-to-test-specification.md
```

**8.7 出力ファイルに何を書くか**

`implementation-summary.md`
1. 実装対象の概要
2. 変更対象ファイル一覧
3. 各変更の目的
4. 設計との対応
5. 未解決事項
6. 設計との差異があればその理由

`source-change-01.md`
1. 初回実装の変更記録
2. 主な差分ポイント
3. 変更ファイルごとの要旨
4. 影響範囲
5. テストで確認すべき箇所

`implementation-to-test-specification.md`
1. テスト対象となる変更点
2. 失敗しやすそうな箇所
3. 重点確認観点
4. 回帰リスク候補
5. テスト設計側への注意

---

**9. テスト仕様書作成エージェント仕様**

**9.1 役割**
設計と実装結果から、ブラウザテストと PHPUnit テストを分けて仕様書化する。

**9.2 対象エージェント**
```text
.github/agents/test-specification.agent.md
```

**9.3 補助 instruction**
```text
.github/instructions/testing/testing-viewpoints.instructions.md
.github/instructions/testing/browser-testing.instructions.md
.github/instructions/testing/phpunit-testing.instructions.md
.github/instructions/workflow/artifact-location.instructions.md
```

**9.4 テンプレート**
```text
.github/docs/templates/test-specification-template.md
```

**9.5 正式入力**
```text
.github/workflow-artifacts/cases/<case-id>/outputs/basic-design/basic-design.md
.github/workflow-artifacts/cases/<case-id>/outputs/detailed-design/detailed-design.md
.github/workflow-artifacts/cases/<case-id>/outputs/implementation/implementation-summary.md
.github/workflow-artifacts/cases/<case-id>/outputs/implementation/source-change-01.md
.github/workflow-artifacts/cases/<case-id>/handoffs/implementation-to-test-specification.md
```

**9.6 入力ファイルに何を書くものか**

`implementation-summary.md`
変更全体の要約

`source-change-01.md`
テスト対象の差分と影響範囲

`implementation-to-test-specification.md`
テストで重点的に拾うべき箇所

**9.7 正式出力**
```text
.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-specification.md
.github/workflow-artifacts/cases/<case-id>/handoffs/test-specification-to-test-execution.md
```

**9.8 出力ファイルに何を書くか**

`test-specification.md`
1. テスト方針
2. 対象範囲
3. 非対象範囲
4. 観点一覧
5. ブラウザテストケース
6. PHPUnit テストケース
7. 前提データ
8. 実施手順
9. 期待結果
10. 証跡取得方法

`test-specification-to-test-execution.md`
1. 実施優先順位
2. 実施時の注意
3. 失敗しそうな箇所
4. 重点ログ確認箇所
5. テスト環境上の注意

---

**10. テスト実施エージェント仕様**

**10.1 役割**
仕様書およびレビュー後 handoff に従ってテストを実施し、再現可能な結果を残す。

このエージェントは、初回テスト実施だけでなく、コードレビュー後の再テスト実施も担当する。

**10.2 対象エージェント**
```text
.github/agents/test-execution.agent.md
```

**10.3 利用 skill**
```text
.github/skills/browser-test-execution/SKILL.md
.github/skills/phpunit-test-execution/SKILL.md
```

**10.4 テンプレート**
```text
.github/docs/templates/test-result-template.md
```

**10.5 正式入力**
```text
.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-specification.md
.github/workflow-artifacts/cases/<case-id>/handoffs/test-specification-to-test-execution.md
.github/workflow-artifacts/cases/<case-id>/handoffs/code-review-to-test-execution.md
```

運用ルール:
1. 初回テスト実施では `test-specification-to-test-execution.md` を使う
2. コードレビュー後の再テストでは `code-review-to-test-execution.md` を使う
3. コードレビュー後の handoff は、再テストの正式入力として扱う
4. コードレビューは終点ではなく、必要な再確認観点を test-execution に返す工程とする

**10.6 入力ファイルに何を書くものか**

`test-specification.md`
実施すべきケース本体

`test-specification-to-test-execution.md`
実施順、注意点、重点確認箇所

`code-review-to-test-execution.md`
コードレビュー指摘を踏まえた再テスト対象、追加観点、重点回帰確認箇所

**10.7 正式出力**
```text
.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-result.md
.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-failures.appendix.md
.github/workflow-artifacts/cases/<case-id>/handoffs/test-execution-to-post-test-fix.md
```

**10.8 出力ファイルに何を書くか**

`test-result.md`
1. 実施概要
2. 実施環境
3. ケース一覧
4. 各ケースの合否
5. 期待結果と実結果
6. 総評
7. 修正要否
8. 初回テストかレビュー後再テストかの区分

`test-failures.appendix.md`
1. 失敗ケースごとの詳細
2. 入力値
3. 実施手順
4. エラーメッセージ
5. ログ
6. スクリーンショット有無
7. 再現性
8. 備考

`test-execution-to-post-test-fix.md`
1. 修正対象候補
2. 再現条件
3. 根本原因の仮説
4. 影響範囲の仮説
5. 修正時の注意

---

**11. テスト後修正エージェント仕様**

**11.1 役割**
失敗原因を特定し、最小十分な修正を行う。

**11.2 対象エージェント**
```text
.github/agents/post-test-fix.agent.md
```

**11.3 正式入力**
```text
.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-result.md
.github/workflow-artifacts/cases/<case-id>/outputs/testing/test-failures.appendix.md
.github/workflow-artifacts/cases/<case-id>/handoffs/test-execution-to-post-test-fix.md
```

**11.4 入力ファイルに何を書くものか**

`test-result.md`
失敗全体の一覧

`test-failures.appendix.md`
原因分析用の詳細

`test-execution-to-post-test-fix.md`
修正時の重点確認点

**11.5 正式出力**
```text
.github/workflow-artifacts/cases/<case-id>/outputs/implementation/source-change-02.md
.github/workflow-artifacts/cases/<case-id>/outputs/implementation/post-test-fix-analysis.md
.github/workflow-artifacts/cases/<case-id>/handoffs/post-test-fix-to-code-review.md
```

**11.6 出力ファイルに何を書くか**

`source-change-02.md`
1. テスト後の修正内容
2. 修正したファイル一覧
3. 修正要点
4. 影響範囲
5. 再確認すべき箇所

`post-test-fix-analysis.md`
1. 原因分析
2. 根本原因
3. 対応内容
4. なぜ再発しにくいか
5. 残留リスク
6. 再テスト推奨箇所

`post-test-fix-to-code-review.md`
1. レビューで見てほしい論点
2. 設計との整合で不安な点
3. 回帰懸念
4. セキュリティ懸念
5. 特に見落としやすい箇所

---

**12. コードレビューエージェント仕様**

**12.1 役割**
設計整合、規約、セキュリティ、副作用、回帰まで含めて総合レビューする。

この工程の目的はレビューで完結することではなく、レビューで新たに見つかった確認観点を test-execution に正式に引き継ぐことを含む。

**12.2 対象エージェント**
```text
.github/agents/code-review.agent.md
```

**12.3 補助 instruction**
```text
.github/instructions/review/review-scope.instructions.md
.github/instructions/workflow/artifact-location.instructions.md
```

**12.4 正式入力**
```text
.github/workflow-artifacts/cases/<case-id>/outputs/basic-design/basic-design.md
.github/workflow-artifacts/cases/<case-id>/outputs/detailed-design/detailed-design.md
.github/workflow-artifacts/cases/<case-id>/outputs/implementation/source-change-02.md
.github/workflow-artifacts/cases/<case-id>/outputs/implementation/post-test-fix-analysis.md
.github/workflow-artifacts/cases/<case-id>/handoffs/post-test-fix-to-code-review.md
```

**12.5 入力ファイルに何を書くものか**

`source-change-02.md`
レビュー対象の変更概要

`post-test-fix-analysis.md`
原因と対応の因果関係

`post-test-fix-to-code-review.md`
レビュー重点論点

**12.6 正式出力**
```text
.github/workflow-artifacts/cases/<case-id>/outputs/review/review-result.md
.github/workflow-artifacts/cases/<case-id>/outputs/review/review-findings.appendix.md
.github/workflow-artifacts/cases/<case-id>/handoffs/code-review-to-test-execution.md
```

**12.7 出力ファイルに何を書くか**

`review-result.md`
1. 総評
2. 指摘概要
3. 重要度別の指摘数
4. 修正要否
5. 再テスト要否
6. 承認可否の判断材料

`review-findings.appendix.md`
1. 指摘一覧
2. 観点分類
3. 根拠
4. 影響範囲
5. 推奨対応
6. 見送り理由があればその記録

`code-review-to-test-execution.md`
1. 再テストが必要なケース
2. 新たに確認すべき観点
3. 重点回帰範囲
4. 指摘ベースの確認項目
5. 次回テスト時の注意

この handoff は、レビュー後に再テストへ戻る際の正式入力です。

重要:
1. コードレビューの handoff 先は test-execution である
2. review-result.md は判定結果であり、code-review-to-test-execution.md は次工程入力である
3. レビューで見つかった追加確認点は、test-execution で再確認されることを前提に扱う
4. このため、コードレビューは品質確認の終点ではなく、再テストを発火させる品質ゲートとして位置付ける

---

**13. レビュー補助エージェント仕様**

すべて agents 直下に置きます。  
すべて `user-invocable: false` を前提にします。

対象ファイル:

```text
.github/agents/review-design-alignment.agent.md
.github/agents/review-coding-rules.agent.md
.github/agents/review-security.agent.md
.github/agents/review-regression.agent.md
.github/agents/review-test-adequacy.agent.md
.github/agents/review-laravel-structure.agent.md
```

役割は次のとおりです。

1. design-alignment
設計と実装の一致確認

2. coding-rules
命名、責務分離、可読性、既存パターン準拠確認

3. security
入力検証、認可、権限、一般的脆弱性確認

4. regression
副作用、デグレ、影響範囲確認

5. test-adequacy
テスト仕様と結果の十分性確認

6. laravel-structure
Middleware、Route、Controller、Request、Service、Repository の責務配置確認

---

**14. 主エージェントと補助エージェントの frontmatter 方針**

**14.1 7体の主エージェント**
推奨:
1. `user-invocable: true`
2. `disable-model-invocation: true`

意味:
1. ユーザーは明示的に選べる
2. 他エージェントが勝手に主工程を起動しない

**14.2 レビュー補助エージェント**
推奨:
1. `user-invocable: false`
2. `disable-model-invocation: false`

意味:
1. ユーザー一覧には出さない
2. コードレビュー親エージェントからのみ使う

**14.3 コードレビュー親エージェント**
推奨:
1. `user-invocable: true`
2. `disable-model-invocation: true`
3. `tools` に `agent` を含める
4. `agents` にレビュー補助エージェント名を列挙する
