---
name: implementation-authority
description: 実装時に詳細設計を主基準とし、基本設計を参考資料として扱うときに使う
applyTo: "app/**/*.php, routes/**/*.php, resources/views/**/*.blade.php, tests/**/*.php"
---
# 実装権威ルール

- 実装判断の主基準は detailed-design.md とする。
- basic-design.md は意図確認用の参考資料として扱う。
- 詳細設計と基本設計に差異がある場合は、まず差異を明示し、独断で都合のよい方を採用しない。
- detailed-design-to-implementation.md に独断禁止事項がある場合は必ず従う。
- 実装中に設計差分が発生した場合は implementation-summary.md に理由を残す。

## 実装時に記録すべきこと

- 変更対象ファイル
- 変更目的
- 設計との対応関係
- 未解決事項
- 設計との差異と理由
- テストで確認すべき論点

## 禁止事項

- 詳細設計を無視して実装を先行させない。
- 未合意の仕様補完をコードに埋め込まない。
- 変更理由を記録せずに影響範囲を広げない。
