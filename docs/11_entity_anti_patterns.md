---
layout: default
---

# JPA / Hibernate アンチパターン集（事故例ベース）→

## この資料の目的

* JPA / Hibernate を **なんとなく使って起きがちな事故** を事前に知る
* エラーが出たときに「見覚えあるやつだ」と気づけるようにする
* 設計レビュー時のチェックリストとして使う

---

## アンチパターン1：EntityをそのままControllerに返す

### 何が起きるか

* JSON変換時に LAZY ロードが走る
* N+1問題が発生
* `LazyInitializationException` が出る

### ダメな例

```java
@GetMapping("/users")
public List<User> getUsers() {
    return userRepository.findAll();
}
```

### なぜダメか

* Controller層でEntityに触る＝永続化コンテキスト外
* 関連Entity取得タイミングが制御不能

### 正解パターン

* DTOに変換して返す
* fetchはService層で制御

---

## アンチパターン2：EAGERで全部解決しようとする

### 何が起きるか

* 予期せぬJOIN
* SQLが巨大化
* パフォーマンス劣化

### ダメな例

```java
@ManyToOne(fetch = FetchType.EAGER)
private User user;
```

### なぜダメか

* 常に取得される
* Repositoryメソッド単位で制御できない

### 正解パターン

* 基本は LAZY
* 必要な場面だけ JOIN FETCH

---

## アンチパターン3：Repositoryにビジネスロジックを書く

### 何が起きるか

* 責務が崩壊
* テストが困難
* 仕様変更に弱くなる

### ダメな例

```java
@Repository
public class UserRepository {
    public void register(User user) {
        // バリデーション
        // ビジネスルール
        em.persist(user);
    }
}
```

### 正解パターン

* Repository = DB操作のみ
* ビジネスロジックはService層

---

## アンチパターン4：save()を呼ばないと更新されないと思っている

### 勘違いポイント

* Entityを変更したら毎回 save() が必要だと思っている

### 実際は

* managed状態なら Dirty Checking で自動UPDATE

### 正しい理解

```java
@Transactional
public void updateUser(Long id) {
    User user = userRepository.findById(id).orElseThrow();
    user.changeName("newName");
    // save() 不要
}
```

---

## アンチパターン5：トランザクション境界が曖昧

### 何が起きるか

* LazyInitializationException
* 意図しないSQL発行

### ダメな例

* Controllerに @Transactional
* Repositoryに @Transactional を乱用

### 正解パターン

* Service層でトランザクションを管理

---

## アンチパターン6：双方向関連を何でも作る

### 何が起きるか

* 無限ループ
* 依存関係が複雑化

### ダメな例

* とりあえず両方向に @OneToMany / @ManyToOne

### 正解パターン

* 基本は単方向
* 必要になってから双方向

---

## アンチパターン7：equals / hashCode を適当に実装

### 何が起きるか

* SetでEntityが壊れる
* 永続化前後で挙動が変わる

### 危険な例

* 全フィールドで equals / hashCode

### 正解パターン

* 識別子（ID）ベース
* または実装しない選択

---

## アンチパターン8：N+1を気合で直そうとする

### ダメな対応

* for文の中で事前アクセス
* EAGERに変更

### 正解パターン

* JOIN FETCH
* DTOクエリ
* バッチフェッチ

---

## アンチパターン9：EntityをDTO代わりに使う

### 何が起きるか

* 画面都合でEntityが歪む
* 設計が破壊される

### 正解パターン

* Entity = ドメイン
* DTO = 表示/API専用

---

## アンチパターン10：JPAを魔法だと思う

### ありがちな思考

* SQL見ない
* ログ見ない

### 正解スタンス

* 発行SQLを常に確認
* 仕組みを理解して使う

---

## まとめ：事故回避チェックリスト

* Entityを外に出していないか
* fetch戦略を意識しているか
* トランザクション境界はServiceか
* Repositoryにロジックを書いていないか
* SQLを確認しているか

👉 このファイルは「やらかした後」ではなく
👉 **やらかす前に読む用**

← [前へ：Spring Data JPA実践](10_spring_data_jpa.html)  
→ [次へ：用語集](99_glossary.html)

🏠 [トップに戻る](index.html)
