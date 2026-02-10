---
layout: default
---

# Spring Data JPA 実践（Repository設計）

## Spring Data JPAとは

- Springが提供する **JPA操作を簡単にするライブラリ**  
- Repositoryインターフェースを作るだけで CRUD が自動生成される
- JPQL / クエリメソッド / カスタムクエリ対応

---

## Repositoryの基本構造

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // メソッド命名でクエリ生成
    List<User> findByName(String name);
}
```

### ポイント

- JpaRepository<Entity, ID>を継承する
- CRUDメソッドは自動生成
- 名前付きクエリ（Query Methods）でJPQL生成可能。

---

## メソッド命名規則（Query Methods）

- findBy + フィールド名 → 例：findByName(String name)
- findBy + 条件 + OrderBy + フィールド名 + Asc/Desc → 例：findByGreaterThanOrderByNameAsc(int age)
- And/Or も利用可→ 例：findByNameAndAge(String name, int age)

---

## 実務っぽい：fetch戦略との組み合わせ

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    @Query("select o from Order o join fetch o.user where o.id = :id")
    Order findWithUserById(@Param("id") Long id);
}
```

- JOIN FETCHを使うことでN+1問題を回避する
- fetch戦略はメソッド単位で制御可能です
- 必要な場面だけJOIN FETCHするのが最善っぽい

---

## Pageable / Sortでページング・ソート

```java
List<User> findByName(String name, Pageable pageable);
```

- ページング対応メソッドも自動生成
- Sortパラメータを付ければソートも可能
- JOIN FETCHとの組み合わせには注意（ページング不可）

---

## カスタムRepositoryの作り方

```java
public interface CustomUserRepository {
    List<User> findCustomUsers();
}

@Repository
public class CustomUserRepositoryImpl implements CustomUserRepository {
    @PersistenceContext
    private EntityManager em;

    @Override
    public List<User> findCustomUsers() {
        return em.createQuery("select u from User u where u.age > 18", User.class).getResultList();
    }
}
```

- Interface + Implで自由にJPQLやQueryDSLを実装する
- 複雑な検索・集計はここで対応することになる

---

## トランザクションとの組み合わせ

- Repositoryは基本**トランザクション境界外**
- Service層に@Transactionalを置く
- readOnlyは参照系メソッドで付与

```java
@Service
public class UserService {

    @Transactional(readOnly = true)
    public List<User> getUsers() {
        return userRepository.findAll();
    }

    @Transactional
    public void createUser(User user) {
        userRepository.save(user);
    }
}
```

---

## 間違いやすいところ

1.**Entityをそのまま返す**
    - ControllerでJSON変換中にLAZYロード→N+1 / LazyInitializationException
    - →DTOに変換を返す
2.**EAGERに頼る**
    - 全部即時取得→パフォーマンス劣化
    - →メソッド単位でfetchを制御
3.**Repositoryにビジネスロックを入れよう**
    - →Service層に集約

---

## DTO変換の実践例

```java
public class UserDto {
    private String name;
    private int age;

    public UserDto(User user) {
        this.name = user.getName();
        this.age = user.getAge();
    }
}
```

```java
List<UserDto> dtos = userRepository.findAll().strean()
                                    .map(UserDto::new)
                                    .collect(Collectors.toList());
```

- Entityをそのまま返さず、DTOに変換する
- JSON変換時の事故を防ぐ

---

## まとめ

- Repository = DB操作担当者
- Service = トランザクションとビジネスロックの担当者
- 必要な時だけ JOIN FETCH / カスタムクエリ
- Entityは外に出さずにDTOを返す
- find / save / deleteの基本CRUDを極力自動利用する

---

← [前へ：JPQL とクエリ設計（JPQL & Query Design）](09_jpql_and_query_design.html)  
→ [次へ：JPA / Hibernate アンチパターン集（事故例ベース）](11_entity_anti_patterns.html)

🏠 [トップに戻る](index.html)
