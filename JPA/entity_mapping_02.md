# 기본 키 매핑

## 기본 키 매핑 어노테이션

- @Id
- @GeneratedValue

---

## @Id

- 내가 직접 id를 설정하고 싶으면 @Id를 붙여주기만 하면 된다.

```java

@Id
private String id;
```

#

## @GeneratedValue

### IDENTITY 전략

- `@GeneratedValue(strategy = GenerationType.IDENTITY)`
- 기본 키 생성을 데이터베이스에 위임한다.
- 주로 MySQL(AUTO_INCREMENT), PostgreSQL, SQL Server, DB2 에서 사용한다.
-
- JPA는 원래 커밋하는시점에 flush가 이루어지지만, 
- IDENTITY 전략의 경우 DB에 INSERT 쿼리를 날려야 Id 값을 알 수 있기 때문에, `em.persist()` 시점에 즉시 쿼리가 나간다.
- 이렇게 전달된 엔티티는 1차캐시에 저장된다.

#

### SEQUENCE 전략

```java
@Entity 
@SequenceGenerator( 
    name = “MEMBER_SEQ_GENERATOR", 
    sequenceName = “MEMBER_SEQ",
    initialValue = 1, allocationSize = 50) 

public class Member { 

    @Id 
    @GeneratedValue(strategy = GenerationType.SEQUENCE, 
            generator = "MEMBER_SEQ_GENERATOR") 
    private Long id; 
```

-  데이터베이스 시퀀스 객체를 이용해서 값을 유일한 순서대로 생성한다.
-  name은 식별자 생성기 이름이고, sequenceName 시퀀스 이름을 지정할 수 있다. 기본 값은 hibernate_sequence이다.
-  
-  IDENTITY 전략과 다른 점은 `em.persist()` 시점에 먼저 시퀀스 객체에서 PK 값만 얻어오도록 DB에 쿼리를 날린다.
-  PK 값을 얻어와서 영속성 컨텍스트에 등록한 후, 실제 트랜잭션 커밋을 하는 시점에 INSERT 쿼리를 날려서 객체를 저장한다. 
-  그렇기 때문에 JPA가 제공하는 버퍼링 기능도 사용이 가능하다.

### SEQUENCE 전략 성능 최적화

- @SequenceGenerator에는 allocationSize라는 속성이 존재하는데, 시퀀스 한 번 호출에 증가하는 수를 의미한다.
- 만약 allocaitonSize가 1이라면 객체를 한 번 저장할 때마다 계속해서 시퀀스 호출이 일어나기 때문에 성능이슈가 일어날 수 있을 것이다.
- allocationSize가 50이면 시퀀스를 한 번 호출할 때마다 DB에 미리 시퀀스를 50만큼 늘려놓고 
- id 시퀀스가 DB 시퀀스와 같아질 때까지 = 51이 될 때까지 시퀀스 호출 없이 메모리에서 하나씩 증가시켜서 사용이 가능하다.

#

### TABLE 전략

```sql
create table MY_SEQUENCES ( 
    sequence_name varchar(255) not null, 
    next_val bigint, 
    primary key ( sequence_name ) 
)
```

```java
@Entity 
@TableGenerator( 
        name = "MEMBER_SEQ_GENERATOR", 
        table = "MY_SEQUENCES", 
        pkColumnValue = “MEMBER_SEQ", allocationSize = 1) 

public class Member { 
    @Id 
    @GeneratedValue(strategy = GenerationType.TABLE, 
                    generator = "MEMBER_SEQ_GENERATOR") 
    private Long id;
```

- 키 생성 전용 테이블을 만들어서 데이터베이스 시퀀스를 흉내내는 전략이다.
- 특정 DB에 의존적이지 않은게 장점이다.
- 성능이 안좋은게 단점.
-
- TABLE 전략도 SEQUENCE 전략과 마찬가지로 allocationSize 속성을 이용해서 최적화가 가능하다.

---
 
## Reference

- [자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)
