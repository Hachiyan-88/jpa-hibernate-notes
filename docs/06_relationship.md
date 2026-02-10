---
layout: default
---

# エンティティ間の関連（@OneToMany / @ManyToOne）

## エンティティの関連とは

エンティティの関連とは、  
**テーブル同士のリレーション（外部キー）を  
Javaオブジェクト同士の参照として表現する仕組み**。

RDB：

- 外部キーで関連付ける

JPA：

- Entityが別のEntityをフィールドとして持つ

---

## 例に使うモデル

### テーブル構造イメージ

- User（ユーザー）
- Order（注文）

関係：

- ユーザーは複数の注文を持つ
- 注文は1人のユーザーに属する

👉 **1対多（OneToMany / ManyToOne）**

---

## @ManyToOne（基本・最重要）

### Order → User（多対一）

```java
@Entity
public class Order {

    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;
}
```

### Point

- **@ManyToOneが持つ外部キーを持つ側**
- DB的にも自然
- 実務ではこちらが主役になることも・・

👉 **まず @ManyToOne を理解するのが最優先**

---

## @JoinColumnとは

```java
@JoinColumn(name = "user_id")
```

- 外部キーのカラム名を指定
- 指定しない場合は自動生成される

### 実務では？

👉 **必ず明示的に書く**(DDLや既存DBとズレる事故を防止する)

---

## @OneToMany（逆側の表現）

### User → Order（1対多）

```java
@Entity
public class User {

    @Id
    @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "user")
    private List<Order> orders = new ArrayList<>();
}
```

### Point2

- mappedByが超重要
- **外部キーは持たない側**
- あくまで「逆から見た表現」

---

## mappedBy とは何か

```java
@OneToMany(mappedBy = "user")
```

意味:

- 「この関連はOrder.userが管理している」
- 自分は**参照専用**

👉 mappedBy がある側は**関連の管理者ではない**

---

## 関連のオーナー（管理者）

JPAでは、**外部キーを管理する側がオーナー。**

今回の場合:

- Order.user → 管理者（オーナー）
- User.orders → 逆側（非オーナー）

⚠️ **更新が反映されるのはオーナー側のみ**

## よくある落とし穴①：片側だけセット

```java
User user = new User();
Order order = new Order();

user.getOrders().add(order);
```

❌ これだけでは不十分。

なぜ？

- 外部キーはOrder側が管理しているから

---

## 正しい関連のセット方法

```java
Order order = new Order();
order.setUser(user);

user.getOrders().add(order);
```

👉 **必ず両方セットする**

実務ではヘルパーメソッドを作ることが多いらしい。

```java
public void addOrder(Order order) {
    orders.add(order);
    order.setUser(this);
}
```

---

## fetch（取得戦略）

### デフォルト値

| アノテーション | デフォルト |
| ---------- | ----- |
| @ManyToOne | EAGER |
| @OneToMany | LAZY |

---

## EAGER の危険性

```java
@ManyToOne(fetch = FetchType.EAGER)
private User user;
```

- 常にJOINされる
- 思わぬSQL増加
- N+1問題の温床

👉 **実務では EAGER は避ける**

---

## 実務の鉄板設定の例

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "user_id")
private User user;
```

- トランザクション外だと例外が出る可能性あり
- 永続化コンテキストが必要

👉 **LAZY = 魔法ではない**

---

## cascade（伝播）

```java
@OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
private List<Order> orders;
```

### できること

- User保存時にOrderも保存
- User削除時にOrderも削除

⚠️ **ALLは強力すぎるので慎重に**

---

## よくある勘違い

- @OneToManyだけあればOK
→ ❌ 管理者は @ManyToOne

- mappedByは外部キー名
→ ❌ フィールド名

- EAGERは便利
→ ❌ 実務では地雷

---

← [前へ：変更検知（Dirty Checking）](05_dirty_checking.md.html)  
→ [次へ：N+1問題（N Plus One Problem）](07_n_plus_one.html)

🏠 [トップに戻る](index.html)
