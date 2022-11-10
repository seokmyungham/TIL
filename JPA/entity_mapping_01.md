# 엔티티 매핑

- 객체와 테이블 매핑
    - @Entity, @Table
- 필드와 컬럼 매핑
    - @Column
- 기본 키 매핑
    - @Id
- 연관관계 매핑
    - @ManyToOne, @JoinColumn

---

## @Entity

- 기본적으로 JPA를 사용해서 테이블과 매핑할 클래스는 @Entity를 붙여주어야 한다.
- @Entity를 붙인 클래스는 기본 생성자를 필수로 갖고 있어야한다. (파라미터가 없는 public 또는 protected 생성자)
- final 클래스, enum, interface, inner 클래스는 사용하면 안된다.
- name 속성으로 JPA에서 사용할 엔티티 이름을 지정할 수 있다.
    - 클래스 이름을 기본값으로 사용한다.
    - 같은 클래스 이름이 없으면 가급적 기본값을 사용하는것이 좋다.

## @Table

- @Table을 사용해서 엔티티와 매핑할 테이블을 지정할 수 있다.
- name 속성으로 매핑할 테이블 이름을 지정할 수 있다.
    - 엔티티 이름을 기본값으로 사용한다.
- 외에도 catalog, schema, uniqueConstraints 속성이 있다.

---

## @Column

- @Column을 이용하면 필드의 다양한 속성을 지정할 수 있다.
- name 속성을 이용하여 필드와 매핑할 테이블의 컬럼 이름을 따로 지정할 수 있다.
    - 객체의 필드 이름을 기본값으로 사용한다.
- nullable(DDL) 속성으로 null 값의 허용 여부를 설정할 수 있다.
- unique(DDL) 속성으로 한 컬럼에 유니크 제약조건을 설정할 수 있다.
- columnDefinition(DDL) 속성으로 데이터베이스 컬럼 정보를 직접 주는 것이 가능하다.
- 외에도 length, precision scale 속성이 있다.

## @Enumerated

- 자바 enum 타입을 매핑할 때 사용한다.
- 속성에는 value가 있는데 value 값으로 EnumType.ORDINAL, EnumType.STRING 이 있다.
-
- 사용할 때 반드시 EnumType.STRING을 사용하도록 하자!
- EnumType.STRING을 사용하면 enum의 이름이 데이터베이스에 저장된다.
- 개발하다가 중간에 enum의 순서가 바뀌어도, 데이터베이스에는 enum의 이름이 저장되어 있기 때문에 데이터 손실을 막을 수 있다.

## @Temporal

- 날짜 타입을 매핑할 때 사용햔다.(java.util.Date, java.util.Calendar)
- 자바 LocalDate, LocalDateTime을 사용한다면 생략이 가능하다.

## @Lob

- 데이터베이스 BLOB, CLOB 타입과 매핑할 떄 사용한다.
- 매핑하는 필드 타입이 문자면 CLOB으로 매핑되고, 나머지는 BLOB으로 매핑된다.

## @Transient

- 매핑을 하기 싫으면 @Transient를 붙이면 된다.
- 주로 메모리상에서만 임시로 어떤 값을 보관하고 싶을 때 사용한다.

---

## Reference

- [자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)
