# API - OnToMany 성능 최적화

## 페치 조인 사용과 페이징 한계 돌파

- 일대일, 다대일 관계와 마찬가지로 일대다 관계에서 페치 조인을 활용하면 여러번 실행되는 SQL을 한 번으로 줄일 수 있다.
- 하지만 컬렉션을 페치 조인할 때는 주의할 점과 몇 가지 한계들이 존재한다.
- 가장 큰 문제로는 일대다 조인을 할 경우에는 데이터베이스 row가 늘어나는 점을 주의해야하고, 페치 조인을 사용하면 페이징이 불가능하다는 한계가 있다.
- [페치 조인의 자세한 내용, 특징과 한계](https://github.com/seokmyungham/TIL/blob/main/JPA/fetch_join_01.md)

---

### 컬렉션 페치 조인의 한계

![](img/tuning_03.PNG)

- Order를 조회할 때 OrderItem의 정보도 함께 필요하다면, 개발자는 페치 조인을 사용하는 방법을 선택할 수도 있다.
- 여기서 Order와 OrderItem은 일대다 관계이며 Order 엔티티는 List\<OrderItem>를 가지고 있다.

```java
public List<Order> findAllWithItem() {
    return em.createQuery("select distinct o from Order o" +
            " join fetch o.member m" + // Order <> Member 다대일
            " join fetch o.delivery d" + // Order <> Delivery 일대일
            " join fetch o.orderItems oi" + // Order <> OrderItem 일대다
            " join fetch oi.item i", Order.class) // OrderItem <> Item 다대일
            .getResultList();
}
```

- 위 처럼 페치 조인을 사용해서 원하는 모든 데이터를 SQL 한 번으로 가져오고 distinct 명령어를 사용하면 데이터의 중복도 제거할 수 있는 JPQL이 완성된다.
- SQL 한 번으로 원하는 데이터를 가져오는 큰 장점은 성능을 최적화하는데 도움이 될 수 있지만,
- 컬렉션을 페치 조인하는 방법은 큰 단점을 갖고있다. 바로 페이징이 불가능하다는 점이다.

![](img/tuning_04.PNG)

- 만약 위 JPQL문에 setFirstResult(), setMaxResult()와 같은 페이징 메소드를 추가해서 페이징을 시도하면
- 하이버네이트는 경고 로그를 남기면서 모든 데이터를 DB에서 읽어오고, DB가 아닌 메모리에서 페이징을 시도한다.
- 그 이유는 일대다를 조인할 경우 데이터의 row수가 늘어나기 때문에, 늘어난 row에서 페이징을 시도해봤자 원하는 값을 얻는 것이 불가능하기 때문이다.
- 이 치명적인 단점때문에 일대다 관계를 페치 조인 하고 페이징을 시도하는 것은 장애로 이어지는 매우 위험한 방법이며 다른 방법을 생각해야 한다.

---

### 페이징과 한계 돌파

