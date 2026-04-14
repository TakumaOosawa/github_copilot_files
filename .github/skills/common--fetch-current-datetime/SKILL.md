---
name: common--fetch-current-datetime
description: 現在日時を取得する際に使用する
---

# 現在日時取得ルール

- #tool:web/fetch を使用して、下記URLから現在日時を取得して、呼び出し元に返す。
  - https://www.worldtimeserver.com/current_time_in_JP.aspx?city=Tokyo&cb={unique_id}
  - {unique_id} は、ランダムに10桁の数字を指定して、毎回異なるURLを生成すること。
- フォーマットは、YYYY-MM-DD HH:mm:ss とする。
  - 例）2024-01-01 12:00:00
- 都度取得した最新日時を記載する。${context}に残っている値を使い回さない。
