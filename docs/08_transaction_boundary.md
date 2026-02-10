# トランザクション境界（Transaction Boundary）

← [トップへ戻る](index.md) | [次へ進む](09_jpql_and_query_design.md) →

## トランザクション境界とは

トランザクション境界とは、  
**「どこからどこまでを1つの処理単位として扱うか」**  
を決める範囲のこと。

JPAでは特に、

- 永続化コンテキストの有効範囲
- 変更検知が動く範囲
- LAZYロードが可能な範囲

を決定づける、超重要概念。

---

## JPAにおけるトランザクションの役割

トランザクションがあることで、次が保証される。

- 一連のDB操作がまとめて成功 or 失敗
- 永続化コンテキストが維持される
- 変更検知（Dirty Checking）が動く
- LAZYロードが可能

👉 **JPAの機能はトランザクション前提**

---

## Springにおける @Transactional

```java
@Transactional
public void updateUserName() {
    User user = entityManager.find(User.class, 1L);
    user.setName("Taro");
}
```

この1行で:

- トランザクション開始
- メソッド終了時に commit
- commit直前に flush
- 永続化コンテキスト破棄

が自動で行われる。

---

## トランザクションと永続化コンテキスト

### 関係性

- トランザクション開始
→ 永続化コンテキスト生成

- トランザクション終了
→ 永続化コンテキスト破棄

👉 **基本は1トランザクション = 1永続化コンテキスト**

---

## トランザクションがないとどうなるか

### 例：変更検知が動かない

```java
public void updateUser() {
    User user = entityManager.find(User.class, 1L);
    user.setName("Taro");
}
```

- @Transactional なし
- トランザクションが存在しない

👉 DBは更新されない

👉 「JPAが壊れている」ように見える典型例

---

## トランザクションがないとどうなるか②

### LAZYロード失敗

```java
User user = userRepository.findById(1L);
System.out.println(user.getOrders().size());
```

- 永続化コンテキストがすでに破棄
- LAZYロード時にDBアクセス不可

👉 LazyInitializationException 発生

---

## よくあるアンチパターン：ControllerでEntity操作

```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userRepository.findById(id);
}
```

- Controllerは基本トランザクション外
- JSON変換時にLAZYアクセス
- 例外 or N+1問題

👉 **Entityをそのまま返さない**

---

## 正しいレイヤ構造と境界

```text
Controller
  ↓
Service  ← @Transactional
  ↓
Repository
```

### Point

- トランザクションは Service層
- Repositoryは薄く
- ControllerはEntityに触らない

---

## readOnlyトランザクション

```java
@Transactional(readOnly = true)
public User findUser(Long id) {
    return userRepository.findById(id);
}
```

### 効果

- 変更検知が抑制される
- 意図しないUPDATE防止
- パフォーマンス向上

👉 **参照系は readOnly=true が基本**

---

## トランザクション境界が広すぎる問題

```java
@Transactional
public void process() {
    // DB処理
    // 外部API呼び出し
    // 重い計算
}
```

問題点:

- DBロック時間が長い
- スケーラビリティ低下

👉 DB操作だけを囲む

---

## トランザクション境界が狭すぎる問題

```java
@Transactional
public User findUser(Long id) {
    return userRepository.findById(id);
}
```

```java
User user = service.findUser(1L);
user.getOrders().size();
```

- Service終了時にトランザクション終了
- その後LAZYアクセス

👉 例外発生

---

## トランザクションとN+1の関係

- トランザクションがある→ LAZYロードが可能
- LAZYロードが可能→ N+1が発生しやすい

👉 N+1はトランザクションがあるから起きる

（＝トランザクションを消しても解決しない）

---

## トランザクション設計の指針

- 更新系 → @Transactional（デフォルト）
- 参照系 → @Transactional(readOnly = true)
- Controllerに@Transactionalを付けない
- EntityはService層まで

---

## よくある勘違い

- @Transactional = 魔法 → ❌ 境界設計が重要

- 付ければ安全 → ❌ 付けすぎも危険

- LAZY例外は悪 → ❌ 設計ミスのサイン
