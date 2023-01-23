# API 개발 시 DTO를 사용해야 하는 이유

### 엔티티(Member)를 외부에 직접 노출하는 경우

```java
@GetMapping("/api/v1/members")
public List<Member> membersV1() {
    return memberService.findMembers();
}
```

- 엔티티의 모든 값들이 외부에 노출된다.
- 엔티티에 @NotEmpty와 같은 API 검증을 위한 로직이 들어가게 된다.
- 엔티티가 변경되면 API 스펙이 변한다
    - 엔티티는 여러 곳에서 사용하기 때문에 스펙이 바뀔 확률이 높다. 
    - 엔티티 수정이 일어날 때 마다 API의 스펙 자체도 바뀌는 것이 가장 큰 문제이다.
- 한 엔티티에 각각의 API를 위한 모든 요청 요구사항을 담기가 어렵다.
- 컬렉션을 직접 반환하게 되면 향후 API 스펙을 변경하기 어렵다.

#

### 엔티티 대신 DTO를 사용하는 경우

```java
@GetMapping("/api/v2/members")
public Result memberV2() {
    List<Member> findMembers = memberService.findMembers();
    
    List<MemberDto> collect = findMembers.stream()
            .map(m -> new MemberDto(m.getName()))
            .collect(Collectors.toList());

    return new Result(collect);
}

@Data
@AllArgsConstructor
static class Result<T> {
    private T data;
}

@Data
@AllArgsConstructor
static class MemberDto {
    private String name;
}
```

- 엔티티와 프레젠테이션 계층을 위한 로직을 분리할 수 있다.
- 엔티티와 API 스펙을 명확하게 분리할 수 있고, 엔티티가 변해도 API 스펙이 변하지 않는다.
    - 엔티티에 수정이 일어나면 API에 영향없이 컴파일 오류로 엔티티 변경사항을 파악할 수 있다.

---

## Reference

- [실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94/dashboard)
