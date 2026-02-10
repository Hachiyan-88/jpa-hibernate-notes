# Entity

## Entityとは

Entityとは、**JPAによって永続化（DB保存）されるオブジェクト**であり、  
**DBのテーブルと1対1で対応するクラス**。

JPAの世界では、Entityがすべての中心になる。

---

## JPA全体像の中でのEntityの位置づけ

JPAは次のような役割分担で成り立っている。

- Entity  
  → **「何を保存するか」を表す**
- EntityManager  
  → **「どう操作するか」を司る**
- 永続化コンテキスト  
  → **Entityを管理する仕組み**
- Hibernate  
  → **実際にSQLを発行するエンジン**

つまり Entity は、

> **「JPAが管理する対象そのもの」**

であり、  
JPAの仕組みはすべて **Entityを正しく扱うため**に存在している。

---

## JavaクラスとDBテーブルの対応関係

| Java | DB |
| --- | --- |
| クラス | テーブル |
| フィールド | カラム |
| インスタンス | レコード |

この対応付けを、**アノテーション**で定義する。

---

## 基本的なEntity定義

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue
    private Long id;

    private String name;
}
