# 参考資料・リンク集（References）

このファイルは、  
JPA / Hibernate を学習・設計する際に参考にした  
**公式ドキュメント・技術記事・解説サイト**をまとめたもの。

「どの記事で何を理解したか」を  
あとからすぐ思い出せるようにする目的。

---

## 公式ドキュメント

### JPA（Jakarta Persistence）

<https://spring.pleiades.io/spring-data/jpa/reference/jpa.html>

- JPA 公式仕様書  
  - Entity / 永続化コンテキスト / ライフサイクルの定義確認用
  - 用語の正確な意味を確認するときに参照

---

### Hibernate ORM Documentation

<https://hibernate.org/orm/documentation/7.2/>

- Hibernate公式ドキュメント  
  - fetch戦略、N+1、バッチフェッチの挙動確認
  - 「なぜそうなるか」を調べるときに使用

---

## 技術記事・ブログ（概念理解）

### 永続化コンテキスト・変更検知

<https://spring-boot.jp/database/overview/382>
<https://qiita.com/haroya01/items/e2fbb38a90a686855b06>

- 永続化コンテキスト解説記事  
  - Dirty Checking の内部動作を理解するために参考
  - スナップショットの考え方が分かりやすい

---

### トランザクション境界

<https://spring.pleiades.io/spring-data/relational/reference/jdbc/transactions.html>
<https://dexall.co.jp/articles/?p=298>
<https://techorgana.com/java/spring_mvc/tx_manage/4955/>

- Spring @Transactional 解説記事  
  - Service層に境界を置く理由の理解
  - readOnly トランザクションの意味確認

---

## N+1問題・パフォーマンス

### N+1問題解説

<https://zenn.dev/manase/scraps/f551b2456be72e>
<https://qiita.com/musenmai/items/e48e5594e6237a57703c>
<https://www.alpha.co.jp/blog/202412_01/>

- N+1問題の典型例まとめ記事  
  - JOIN FETCH が必要になるケース整理
  - EAGERが危険な理由の確認

---

### fetch戦略・JOIN FETCH

<https://zenn.dev/dev_yoon/articles/985691468ffc76>

- JOIN FETCH 詳細解説記事  
  - DISTINCT の意味
  - ページングと相性が悪い理由

---

## JPQL / クエリ設計

### JPQL基礎

<https://spring-boot.jp/database/overview/75>
<https://spring-boot.jp/database/query-methods/77>

- JPQL入門記事  
  - SQLとの違いを整理
  - JOIN / WHERE の書き方確認

---

### 実務向けクエリ設計

<https://qiita.com/krile136/items/235c543eff547a4cfe50>
<https://zenn.dev/smartshopping/articles/4763ab1247a258>

- Repository設計・クエリ分離の記事  
  - メソッド単位で取得戦略を変える考え方
  - find / findWithXXX の命名指針

---

## 使い方メモ

- 新しい記事を読んだらここに追記
- URLだけでなく「何の理解に使ったか」を1行書く
- glossary とセットで育てる

## 参考

<https://zenn.dev/crsc1206/books/d8166194fd58f2/viewer/744fe5>
<https://spring-boot.jp/database/overview/65>
<https://chigai2.fromation.co.jp/archives/37044>
