
# EntityManager

## EntityManagerとは？

EntityManager　は、
**JPAでDB操作を行うための中心的な存在**。

Entityを直接DBに保存・取得しているように見えて、
実際には **EntityManager を通して操作**　している。

---

## JPA全体の中での役割

- Entity
    →　永続化される対象
- EntityManager
    →　Entityを操作する窓口
- 永続化コンテキスト
    →　Entityの管理領域

EntityManager　は
**永続化コンテキストを操作するためのAPI**とも言えると思う。

---

## 主な役割

- Entityの永続化（保存）
- Entityの検索（取得）
- Entityの更新
- Entityの削除
- 永続化コンテキストの管理

---

## 主なメソッド

| メソッド | 内容 |
| --------- | ---- |
| persist | 新規Entityを永続化 |
| find | 主キーでEntityを取得 |
| merge | Entityの状態を更新 |
| remove | Entityを削除 |
| flush | SQLを即時発行 |

---

## find の特徴

永続化コンテキスト内にEntityが存在すれば
→ **SQLは発行されない**

同じIDのEntityは
→ **必ず同一インスタンスになる**

```java
User u1 = entityManager.find(User.class, 1L);
User u2 = entityManager.find(User.class, 1L);

// u1 == u2
```

---

## なぜ同一インスタンスになるのか

永続化コンテキストが
**「ID → Entity」の対応関係を管理している**ため

これにより

- 無駄なSELECTを防ぐ
- 変更検知が正しく動く

---

## merge：Entityを統合する（注意が必要）

```java
- User detachedUser = new User();
- User managedUser = entityManager.merge(detachedUser);
```

---

## merge の注意点

戻り値が **managed状態のEntity**

引数のEntityは **管理されないまま**

状態をコピーした **別インスタンス** が管理対象になる

```java

detachedUser != managedUser

```

---

## 注意すべき理由

安易に使うと

どのEntityが管理されているか分からなくなる

更新目的であれば
**find → 値変更（変更検知）** の方が安全で分かりやすい

---

## remove：Entityを削除する

```java
entityManager.remove(user);
```

### remove の注意点

**managed状態のEntityのみ** 削除可能

呼び出し時点ではDELETE文は即時発行されない

Entityは **removed状態** になり、
トランザクション終了時にDELETE文が発行される

---

## flush：SQLを即時発行する

```java
entityManager.flush();
```

永続化コンテキストの変更内容をDBに反映する

トランザクション自体は終了しない

通常の業務処理では明示的に呼ぶことは少ない

---

## EntityManagerとトランザクション

### 基本ルール

EntityManagerの操作は
**トランザクション内で行うのが前提**

トランザクション外では

変更検知が動かない

永続化の保証がない

---

## Springでの利用例

```java
@Transactional
public void updateUser() {
    User user = entityManager.find(User.class, 1L);
    user.setName("Taro");
}
```

@Transactional により

メソッド開始時にトランザクション開始

終了時にcommit

commit時に変更検知が実行される

---

## よくある勘違い

EntityManager = DB接続
→ ❌ 違う（Entity管理と操作の窓口）

merge = 更新専用メソッド
→ ❌ managed状態にするための操作

persistしたら即INSERTされる
→ ❌ タイミングは保証されない

---

## 基本的な使い方

```java
User user = new User("Taro");
entityManager.persist(user);
```
