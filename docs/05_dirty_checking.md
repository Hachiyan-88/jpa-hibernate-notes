
# 変更検知（Dirty Checking）

## 変更検知とは

変更検知（Dirty Checking）とは、  
**managed状態のEntityに対する変更をJPAが自動で検知し、  
トランザクション終了時にUPDATE文を発行する仕組み**。

JPAで  
「UPDATE文を書いていないのにDBが更新される」  
理由の正体がこれ。

---

## なぜ変更検知が必要なのか

もし変更検知がなければ、毎回こう書く必要がある。

```java
entityManager.update(user);
```

JPAでは、

- Entityを取得
- 値を変更
- 何もしない

これだけで更新される

👉 **更新処理を意識しなくていい**

👉 **業務ロジックに集中できる**

---

## 変更検知が動く基本例

```java
@Transactional
public void updateUser() {
    User user = entityManager.find(User.class, 1L);
    user.setName("Taro");
}
```

- UPDATE文は書いていない。
- それでもトランザクション終了時にはUPDATEが発行される。

---

## 変更検知の仕組み（内部動作）

変更検知は次の流れで行われている。

1. Entity取得時

    →　**スナップショット（初期状態）を保存**

2. Entityのフィールドが変更される。
3. トランザクション終了時（commit）
4. 現在の状態とスナップショットを比較
5. 差分があればUPDATEを発行

👉 比較は **永続化コンテキスト内** で行われる。

---

## 重要な前提条件

変更検知が動くためには、つぎの条件が必要。

- Entityが**managed**状態である
- **トランザクション内** である
- 永続化コンテキストが有効である

どれか1つでも欠けると動かない

---

## 動かない例①：トランザクション外

```java
public void updateUser() {
    User user = entityManager.find(User.class, 1L);
    user.setName("Taro");
}
```

- @Transactionalがない
- トランザクションが存在しない

👉 **変更検知は動かない**

👉 DBは更新されない

---

## 動かない例②：detached状態

```java
User user = entityManager.find(User.class, 1L);
entityManager.detach(user);

user.setName("Taro");
```

- Entityが永続化コンテキストから外れてる
- managed状態ではない

👉 変更検知は行われない

---

## flush と変更検知の関係

```java
entityManager.flash();
```

- 永続化コンテキストの変更内容をDBに反映する
- トランザクションは終了しない
- 差分があればUPDATEが発行しない

### ポイント

- flush = commit ではない
- commit時には**必ず内部でflushが行われる**

---

## なぜ「全カラムUPDATE」になることがあるのか

Hibernateのデフォルトでは、

- 差分が1項目でも
- **全カラムUPDATE**が発行されることがある

理由：

- Entity全体を1つの状態として管理しているため

※設定次第で「差分UPDATE」にもできるが、
まずは**挙動を理解することを優先しよう**

---

## merge と変更検知の関係

```java
User managedUser = entityManager.merge(detachedUser);
managedUser.setName("Taro");
```

- mergeによりmanagedなEntityが生成される。
- 変更検知が働くのは**戻り値のEntity**

⚠️ 引数のEntityは変更検知されない点に注意。

---

## 変更検知のメリットと注意点

### メリット

- UPDATE処理を書かなくていい
- バグを生みにくい
- 業務ロジックがシンプルになる

### デメリット

- 意図しない変更もUPDATEされてしまう
- EntityをDTOクラスの代わりに使ってしまうと、事故りやすい
- トランザクション境界を理解しておかないと、挙動が分かりにくい

---

## よくある勘違い

-setterを呼んだ瞬間にUPDATEされてしまう
→ ❌ commit時

- findすれば必ず変更検知される
→ ❌ managed + トランザクション必須

- mergeすれば安全
→ ❌ 状態管理が分かりづらくなる
