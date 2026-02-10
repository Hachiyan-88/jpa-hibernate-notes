---
layout: default
---

# 永続化コンテキスト（Persistence Context）

## 永続化コンテキストとは？

永続化コンテキストとは、
**JPAがEntityを管理するためのメモリ上の領域（箱）**。

EntityManagerは、
この永続化コンテキストを通してEntityを管理・操作している。

---

## なぜ永続化コンテキストが必要なの？

永続化コンテキストがあることで、以下のことが可能になる。

- 同じEntityを何度もDBから取得しない。
- 同一Entityを同一インスタンスとして扱える。
- Entityの変更を自動で検出できる（変更検知）

👉 JPAの「魔法」は、
**すべての永続化コンテキストが前提** になっている。

---

## EntityManagerとの関係

- EntityManager
    →　永続化コンテキストを操作する窓口
- 永続化コンテキスト
    →　Entityを実際に管理する場所

```java
User user = entityManager.find(User.class, 1L);
```

この時EntityManagerは、

1. 永続化コンテキストにEntityがあるか確認
2. なければDBから取得する
3. 永続化コンテキストに登録して返す

---

## 同一Entityは同一インスタンスになる

```java
User u1 = entityManager.find(User.class, 1L);
User u2 = entityManager.find(User.class, 1L);
```

```java
u1 == u2 //true
```

**なぜ？**

- 永続化コンテキストは「**主キー→Entityインスタンス**」を1対1で管理している。
- すでに管理中のEntityがあれば、それを再利用する。

---

## 永続化コンテキストとSQL発行

```java
User user = entityManager.find(User.class, 1L);
User user2 = entityManager.find(User.class, 1L);
```

- 最初のfind
    →SELECTが発行される

- 2回目のfind
    →**SQLは発行されない**

👉 DBアクセス削減につながる重要な仕組みらしい

---

## Entityの状態（ライフサイクル）

Entityは、永続化コンテキストとの関係によって状態を持つ

| 状態 | 説明 |
| -------- | ------------- |
| new | ただのJavaオブジェクト |
| managed | 永続化コンテキストで管理中 |
| detached | 管理から外れた状態 |
| removed | 削除予定状態 |

---

## managed状態が重要な理由とは？

JPAの便利機能は、
**managed状態のEntityにしか効かない**

1. 変更検知（Dirty/Checking）

2. 同一インスタンス保証

3. 自動UPDATE / DELETE

```java
User user = entityManager.find(User.class, 1L);
user.setName("Taro");
```

→UPDATE文を書いていないが、DBは更新される

---

## 永続化コンテキストと変更検知（概要）

- Entity取得時にスナップショットを保持する
- トランザクション終了時に差分チェックする
- 差分があればUPDATEを発行する

※詳細は 05_dirty-checking.md で扱います

---

## 永続化コンテキストが切れるタイミング

- トランザクション終了時
- EntityManagerがクローズされた場合

その結果：

- Entityは **detached状態** になる
- 変更検知は動かない
- Lazyロードは失敗する可能性がある

---

## よくある勘違い

- 永続化コンテキスト = DB
→ ❌ メモリ上の管理領域

- findするたびにSQLが飛ぶ
→ ❌ 管理中なら飛ばない

- Entityは常に管理されている
→ ❌ managed状態のみ

<div class="warning-box">
<strong>⚠️ WARNING</strong>  
永続化コンテキストの状態と  
DB の状態は常に一致しているとは限りません。

「今DBはこうなっているはず」と思い込むと、  
挙動が理解できなくなります。
</div>


---

← [前へ：EntityManager](03_entity_manager.html)  
→ [次へ：変更検知（Dirty Checking）](05_dirty_checking.html)

🏠 [トップに戻る](index.html)
