# N+1問題（N Plus One Problem）

← [トップへ戻る](index.md) | [次へ進む](08_transaction_boundary.md) →

## N+1問題とは

N+1問題とは、  
**1回のクエリのつもりが、実際には N+1 回のSQLが発行されてしまう問題**。

- 最初に 1回（親を取得）
- その後、N回（子を1件ずつ取得）

👉 JPA / Hibernate を使っていると  
**意識しないとほぼ確実に踏む**。

---

## 典型的な発生例

### 前提の関連

- User：親
- Order：子
- 関係：User 1人に Order が複数

```java
@OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
private List<Order> orders;
```

---

## 問題が起きるコード

```java
List<User> users = entityManager
        .createQuery("select u from User u", User.class)
        .getResultList();

for (User user : users) {
    System.out.println(user.getOrders().size());
}
```

---

## 実際に発行されるSQL

1️⃣ User一覧取得

```sql
select * from user;

```

2️⃣ UserごとにOrder取得

```sql
select * from order where user_id = 1;
select * from order where user_id = 2;
select * from order where user_id = 3;
...
```

👉 **UserがN件なら、Order取得SQLがN回**

これが **N + 1問題**。

---

## なぜ起きるのか

原因はこの3点の組み合わせ。

- @OneToMany が LAZY
- forループで関連Entityにアクセス
- 永続化コンテキストが有効

👉 「必要になった瞬間に取得する」

👉 結果として **大量のSQL**

---

## EAGERにすれば解決する？

```java
@OneToMany(fetch = FetchType.EAGER)
private List<Order> orders;
```

❌ **おすすめしない**

理由:

- 常にJOINされる
- 不要なデータまで取得
- 別のN+1やパフォーマンス劣化の原因

👉 **EAGERは解決策ではない**

---

## 解決策①：JOIN FETCH（王道）

```java
List<User> users = entityManager
        .createQuery(
            "select u from User u join fetch u.orders",
            User.class
        )
        .getResultList();
```

### 効果

- User と Order を **1回のSQLで取得**
- N+1が発生しない

---

## JOIN FETCH の注意点

### 重複行問題

```sql
User × Order の件数分、結果行が増える
```

Hibernateは内部でUserをまとめてくれるが、

- ページング（LIMIT/OFFSET）と相性が悪い
- 大量データではメモリ負荷

---

## 解決策②：必要な場面だけJOIN FETCH

👉 **「一覧画面」など必要なときだけ使う**

- 常にJOIN FETCHしない
- Repository / Query単位で制御

---

## 解決策③：バッチフェッチ（Hibernate）

```properties
hibernate.default_batch_fetch_size=100
```

または

```java
@BatchSize(size = 100)
```

### どういうことが可能になる？

- 1件ずつ → まとめてIN句で取得
- 完全解消ではないが **緩和できる**

---

## JOIN FETCH vs バッチフェッチ

| 観点 | JOIN FETCH | バッチフェッチ |
| ----- | ---------- | ------- |
| SQL回数 | 最小 | 減る |
| ページング | 苦手 | 得意 |
| 取得制御 | 明示的 | 自動 |
| 実務使用 | ◎ | ○ |

👉 **基本は JOIN FETCH**

👉 補助的に バッチフェッチ

---

## よくある勘違い

- LAZYなら安全 → ❌ アクセスした瞬間に地雷
- EAGERなら解決 → ❌ 別の問題を生む
- findは1回だから大丈夫 → ❌ ループ内アクセスが危険

---
