# 고급 매핑

## 상속관계 매핑

- 관계형 데이터베이스는 상속 관계라는 것이 존재하지 않는다.  
- RDB의 슈퍼타입 서브타입 관계라는 모델링 기법이 객체 상속과 유사하다.  

### 상속관계 매핑이란 객체의 상속 관계와 DB의 슈퍼, 서브타입 관계를 서로 매핑하는 것을 말한다.

#

슈퍼, 서브타입 논리모델을 실제 물리 모델로 구현하는 방법은 세가지가 있다.

### @Inheritance(strategy = InheritanceType.XXX)

- JOINED : 조인전략
- SINGLE_TABLE : 단일 테이블 전략
- TABLE_PER_CLASS : 구현 클래스마다 테이블 전략

---

## 조인 전략

![](img/advanced_mapping_01.PNG)

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;
}
```

```java
Entity
public class Movie extends Item {

    private String director;
    private String actor;
}
```

- `@Inheritance(strategy = InheritanceType.JOINED)` 을 사용한 부모 클래스를 자식에 물려주면 정교화된 조인 DB 모델을 구현할 수 있다.

```java
Movie movie = new Movie();
movie.setName("웰시코기");
movie.setDirector("ham");
movie.setActor("corgi");
movie.setPrice(100);

em.persist(movie);

em.flush();
em.clear();

tx.commit();
```

![](img/advanced_mapping_02.PNG)

- 단지 Movie에만 값을 넣어도 JPA가 알아서 INSERT 쿼리를 Item 테이블에도 날려준다.


![](img/advanced_mapping_03.PNG)

#

## @DiscriminatorColumn

- `@DiscriminatorColumn` 어노테이션을 부모 클래스에 사용하면 테이블에 DTYPE이 생기면서, 어느 자식 타입의 데이터인지 확인할 수 있다.  
- name 속성으로 따로 컬럼 명을 지정할 수 있으며, 기본값은 엔티티 명이다.

![](img/advanced_mapping_04.PNG)  

- `@DiscriminatorValue(“XXX”)` 을 자식 클래스에 사용해서 어떤 값으로 데이터에서 나타낼지 설정이 가능하다.

#



