---

layout: default

---

# JPA   /   Hibernate　学習メモ

このサイトは、
**JPA / Hibernate を「なんとなく」から「ちょっと分かる」にするための学習メモ**です。

- 公式ドキュメントや参考ページを基に自分なりに整理
- 「なんでそう動くのか」を自分の言葉で説明できるようになることを重視
- あとから読み返しても理解できる構成を意識しています

※個人の学習メモですが、
同じように悩んでいる方の参考になれば嬉しいなと思います。

---

## 目的

- JPAの挙動を**なぜそう動くのか説明できるようになる**
- エラーが出た場合に**原因を推測できるようになる**
- 実務で怖くならないように・・・

---

## 対象読者

- JPAをこれから使い始める人
- なんとなく使っていて挙動が不安な人
- Hibernateでハマった経験がある人

---

## 目次

| No | 内容 |
| ---- | ------ |
| 01 | [JPA全体像](01_jpa_overview.md) |
| 02 | [Entity](02_entity.md) |
| 03 | [EntityManager](03_entity_manager.md) |
| 04 | [永続化コンテキスト](04_persistence_context.md) |
| 05 | [Dirty Checking](05_dirty_checking.md) |
| 06 | [エンティティ関連](06_relationship.md) |
| 07 | [N+1問題](07_n_plus_one.md) |
| 08 | [トランザクション境界](08_transaction_boundary.md) |
| 10 | [Spring Data JPA](10_spring_data_jpa.md) |
| 11 | [アンチパターン集](11_entity_anti_patterns.md) |
| 99 | [用語集](99_glossary.md) |

---

## スタンス

- EntityをDTOとして使わない
- fetch戦略を意識する
- トランザクション境界はService層
- SQLは必ず確認する

---

## 進め方

- 各mdは1テーマ1ファイル
- 図やコードは積極的に追加
- 実務ではまった場合は追記していく

---

## ライセンス

MIT License
