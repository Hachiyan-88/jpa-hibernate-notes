---
layout: default
---

# JPQL とクエリ設計（JPQL & Query Design）

## JPQLとは

JPQL（Java Persistence Query Language）とは、  
**Entityを対象にしたクエリ言語**。

- SQL：テーブル・カラムを操作
- JPQL：クラス・フィールドを操作

👉 **DBではなく、オブジェクト視点で書く**

---

## SQLとの違い（超重要）

### SQL

```sql
select * from user u where u.name = 'Taro';
```

### JPQL

```java
select u from User u where u.name = 'Taro'
```

違い:

- テーブル名→Entity名
- カラム名→フィールド名

👉 **SQL脳のまま書くと事故る**

---

## JPQLの基本構文

```java
select エイリアス
from Entity名 エイリアス
where 条件
```

```java
select u
from User u
where u.id = 1
```

---

## Entity間の関連は JOIN でたどる

### 前提

- User ↔ Order（1対多）

```java
select o
from Order o
join o.user u
where u.name = 'Taro'
```

👉 **外部キーは書かない**

👉 **オブジェクトの関連をたどる**

---

## JOIN と JOIN FETCH の違い

### JOIN（通常）

```java
select u
from User u
join u.orders o
```

- JOINはする
- Orderは初期化されない
- 後でアクセスするとLAZYロード

---

## JOIN FETCH（取得目的）

```java
select u
from User u
join fetch u.orders
```

- JOINしつつ取得
- Orderが即ロードされる
- N+1問題の回避

👉 **JOIN FETCH = 取得戦略**

---

## JOIN FETCHの設計指針

- 一覧画面：使うことが多い
- 詳細画面：ほぼ必須
- 更新処理：不要なことが多い

👉 **「どこで必要か」を意識する**

---

## WHERE句の注意点

```java
select u
from User u
where u.orders.size > 0
```

⚠️ 落とし穴：

- size() はサブクエリになる
- パフォーマンス劣化しやすい

代替案：

```java
select distinct u
from User u
join u.orders o
```

---

## DISTINCT の役割（JPQL特有）

```java
select distinct u
from User u
join fetch u.orders
```

- SQLのDISTINCTとは少し意味が違う
- Entityの重複をHibernateが除外

👉 JOIN FETCH + distinct は定番らしい

---

## ページングとJOIN FETCH

```java
select u
from User u
join fetch u.orders
```

⚠️ 問題点：

- ページング不可 or 不正確
- Hibernateが警告を出すことがある

👉 **ページングするならJOIN FETCHは避ける方がよさそう**

---

## ページングの実務解決パターン

### パターン①：IDだけ先に取る

```java
select u.id
from User u
```

→ IDsを使って詳細取得

### パターン②：バッチフェッチ併用

```properties
hibernate.default_batch_fetch_size=100
```

- JOIN FETCHしない
- N+1を緩和
- ページングと相性良し

---

## パラメータバインド（必須）

```java
select u
from User u
where u.name = :name
```

```java
query.setParameter("name","Taro");
```

👉 SQLインジェクション対策

👉 可読性向上

---

## 動的クエリの考え方

JPQLをif文で組み立てるのは地獄。

👉 実務では：

- Criteria API
- QueryDSL
- Specification

※ 今は「JPQLで設計思想を理解する」が目的。

---

## Repository設計の指針

- findById → 単純取得
- findWithOrders → JOIN FETCH
- findPage → ページング重視

👉 **メソッド名に意図を出す**

---

## よくあるアンチパターン

- 全部JOIN FETCH → ❌ 重い・ページング不可

- ControllerでJPQL → ❌ 境界崩壊

- Entityをそのまま返す → ❌ N+1 & LAZY地雷

---

← [前へ：トランザクション境界（Transaction Boundary）](08_transaction_boundary.html)  
→ [次へ：Spring Data JPA 実践（Repository設計）](10_spring_data_jpa.html)

🏠 [トップに戻る](index.html)
