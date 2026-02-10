---
layout: default
---

# JPAとは何か

## 一言で表すなら・・・

JPA（Java Persistence API）とは
**JPA = JavaでDB（データベース）を扱うための「仕様（ルール）」**

## JPAとHibernateの関係性

- JPA : 仕様
- Hibernate : JPAの実装

例えると:

- JPA = 運転のルール
- Hibernate = 実際の車

## なぜJPAが必要なのか？

- SQLを大量に記述しなくて済む
- JavaのオブジェクトとDBを自然に対応させることが可能
- CRUDが楽になる

## JPAでやりたいこと

- クラス　⇔　テーブル
- フィールド　⇔　カラム
- オブジェクト　⇔　レコード

---

<div class="warning-box">
<strong>⚠️ WARNING</strong>  
JPA を使っていても、  
「SQL がどう発行されるか」を意識しないと  
パフォーマンス問題や予期しない挙動につながります。

「SQL を書かなくていい」＝「SQL を知らなくていい」ではないです。

</div>

---

→ [次へ：Entity](02_entity.html)

🏠 [トップに戻る](index.html)
